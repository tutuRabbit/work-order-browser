<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container>
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item
              label="产品编码"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="lastProductCode"
              :rules="[{ required: true, message: '请填写产品编码!' }]"
            >
              <a-select
                :disabled="formDisabled"
                show-search
                :filterOption="filterOption"
                v-model="model.lastProductCode"
                placeholder="请选择"
                style="width: 50%"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option
                  @click="fieldsChange(item)"
                  v-for="(item, index) in SupplierOptions"
                  :key="index"
                  :value="item.code"
                >
                  {{ item.code }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.lastProductCode">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="lastProductName">
              {{ model.lastProductName }}
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.lastProductCode">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="lastProductSpec">
              {{ model.lastProductSpec }}
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.lastProductCode">
            <a-form-model-item label="单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="lastProductUnitName">
              {{ model.lastProductUnitName }}
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input
                style="width: 50%"
                :disabled="formDisabled"
                v-model="model.remark"
                placeholder="请输入备注"
              ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="子产品列表" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fields">
              <template>
                <a-button v-if="!formDisabled" class="new-btn" @click="selectSparePart" type="primary">
                  <a-icon type="plus" />
                  选择产品
                </a-button>
                <j-vxe-table
                  ref="prodRef"
                  row-number
                  row-selection
                  keep-source
                  :height="200"
                  :loading="prod.loading"
                  :dataSource="prod.dataSource"
                  :columns="prod.columns"
                  style="margin-top: 8px;"
                >
                  <template v-slot:action="props">
                    <a-popconfirm title="确定删除吗？" v-if="!formDisabled" @confirm="handleDL(props)">
                      <a>删除</a>
                    </a-popconfirm>
                  </template>
                </j-vxe-table>
              </template>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
    <!-- 选择产品 -->
    <j-modal title="选择产品" width="80%" :visible="prodVis" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <!--      <div class="table-page-search-wrapper">-->
      <!--        <a-form layout="inline" @keyup.enter.native="searchEquipment">-->
      <!--          <a-row :gutter="24">-->
      <!--            <a-col :span="6">-->
      <!--              <a-form-item label="产品编码">-->
      <!--                <a-input placeholder="请输入" v-model.trim="prodParam.code"></a-input>-->
      <!--              </a-form-item>-->
      <!--            </a-col>-->
      <!--            <a-col :span="6">-->
      <!--              <a-form-item label="产品名称">-->
      <!--                <a-input placeholder="请输入" v-model.trim="prodParam.productName"></a-input>-->
      <!--              </a-form-item>-->
      <!--            </a-col>-->
      <!--            <a-col :span="6">-->
      <!--              <a-form-item label="产品规格">-->
      <!--                <a-input placeholder="请输入" v-model.trim="prodParam.productNorms"></a-input>-->
      <!--              </a-form-item>-->
      <!--            </a-col>-->

      <!--            <a-col :span="4">-->
      <!--              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">-->
      <!--                <a-button type="primary" @click="searchEquipmen@t" icon="search">查询1</a-button>-->
      <!--                <a-button type="primary" @click="searchEquipmentReset" icon="reload" style="margin-left: 8px"-->
      <!--                >重置</a-button-->
      <!--                >-->
      <!--              </span>-->
      <!--            </a-col>-->
      <!--          </a-row>-->
      <!--        </a-form>-->
      <!--      </div>-->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: prodSelectedRowKeys, onChange: prodOnSelectChange }"
        class="j-table-force-nowrap"
        style="margin-top: 10px"
        @change="handleEquipmentTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { mixinDevice } from '@/utils/mixin'

export default {
  name: 'SysMaterialForm',
  components: {},
  mixins: [JeecgListMixin, mixinDevice],
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
      SupplierOptions: [],
      listCont: [],
      recordList: [],
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 2 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 21 }
      },
      confirmLoading: false,
      editDisabled: false,
      validatorRules: {},
      url: {
        list: '/longxianda/sysProduct/list',
        add: '/longxianda/sysMaterial/add',
        edit: '/longxianda/sysMaterial/edit',
        prodList: '/longxianda/sysProduct/list', //产品定义分页
        queryById: '/longxianda/sysMaterial/queryById'
      },
      columns: [
        {
          title: '产品编码',
          align: 'center',
          dataIndex: 'code'
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productNorms'
        },
        {
          title: '库存单位',
          align: 'center',
          dataIndex: 'stockPiece'
        },
        {
          title: '产品属性',
          align: 'center',
          dataIndex: 'productProperty'
        },
        {
          title: '工艺路线',
          align: 'center',
          dataIndex: 'craftRouteName'
        },
        {
          title: '最大库存',
          align: 'center',
          dataIndex: 'maxStock'
        },
        {
          title: '最小库存',
          align: 'center',
          dataIndex: 'minStock'
        },
        {
          title: '安全库存',
          align: 'center',
          dataIndex: 'securityStock'
        },
        {
          title: '库存数量',
          align: 'center',
          dataIndex: 'stockNumber'
        }
      ],
      treeData: [],
      prodSelectedRowKeys: [],
      prodSelectionRows: [],
      prodVis: false,
      prod: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '60px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '产品编码',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'code'
          },
          {
            title: '产品名称',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'productName'
          },
          {
            title: '产品规格',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'productNorms'
          },
          {
            title: '单位用量',
            align: 'center',
            width: '18%',
            type: FormTypes.inputNumber,
            key: 'useNum',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '库存单位',
            align: 'center',
            type: FormTypes.normal,
            width: '10%',
            key: 'stockPiece'
          },
          {
            title: '产品属性',
            align: 'center',
            type: FormTypes.normal,
            width: '10%',
            key: 'productProperty'
          },
          {
            title: '工艺路线',
            align: 'center',
            width: '10%',
            type: FormTypes.normal,
            key: 'craftRouteName'
          }
        ]
      },
      loading: false,
      dataSource: [],
      ipagination: {
        current: 1,
        pageSize: 5,
        pageSizeOptions: ['5', '10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      prodParam: {},
      superFieldList: []
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.loadList()
    this.getSuperFieldList()
  },
  methods: {
    filterOption(value, option) {
      return option.componentOptions.children[0].text.indexOf(value) >= 0
    },
    // 获取产品列表
    loadList() {
      getAction('/longxianda/sysProduct/list', { pageNo: 1, pageSize: 99999 }).then(res => {
        if (res.success) {
          let arr = [...res.result.records]
          this.SupplierOptions = arr
        }
      })
    },
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
      let id = props.row.id
      for (var i = 0; i < this.prod.dataSource.length; i++) {
        if (this.prod.dataSource[i].id == id) {
          this.prod.dataSource.splice(i, 1)
        }
      }
      // console.log(' this.prod.dataSource', this.prod.dataSource)
    },
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.prodLoad()
    },
    handleCancel() {
      this.prodVis = false
      this.prodParam = {}
      // this.table.dataSource = []
      this.prodSelectedRowKeys = []
      this.prodSelectionRows = []
    },
    handleOk() {
      this.prod.dataSource = this.prodSelectionRows
      this.handleCancel()
    },
    async handleSuperQuery1(params, matchType) {
      await this.handleSuperQuery(params, matchType, true)
      console.log('2')
    },
    prodOnSelectChange(prodSelectedRowKeys, prodSelectionRows) {
      this.prodSelectedRowKeys = prodSelectedRowKeys
      this.prodSelectionRows = prodSelectionRows
    },
    searchEquipment() {
      this.prodLoad(1)
      this.prodSelectedRowKeys = []
      this.prodSelectionRows = []
    },
    selectSparePart() {
      this.prodLoad()
      this.prodVis = true
    },
    searchEquipmentReset() {
      this.prodParam = {}
      this.prodLoad(1)
    },
    prodLoad(arg) {
      if (!this.url.prodList) {
        this.$message.error('请设置url.prodList属性!')
        return
      }
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = Object.assign(this.prodParam)
      params.model = 0
      params.pageNo = this.ipagination.current
      params.pageSize = this.ipagination.pageSize
      this.loading = true
      getAction(this.url.prodList, params)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    delNext(index) {
      var listCont = this.listCont
      listCont.splice(index, 1)
    },
    // 产品信息监听
    fieldsChange(info) {
      // console.log('info', info)
      let that = this
      // 产品名称
      that.model.lastProductName = info.productName
      // 产品规格
      that.model.lastProductSpec = info.productNorms
      // 单位
      that.model.lastProductUnitName = info.stockPiece
    },
    lineInfosChange(info, index) {
      this.listCont[index].code = info.code
      this.listCont[index].procedureId = info.id
      this.listCont[index].report = info.outputRate
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'code', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productNorms', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockPiece', text: '库存单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productProperty', text: '产品属性', dictCode: '' })
      fieldList.push({ type: 'string', value: 'craftRoute', text: '工艺路线', dictCode: '' })
      fieldList.push({ type: 'int', value: 'maxStock', text: '最大库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'minStock', text: '最小库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'securityStock', text: '安全库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'stockNumber', text: '库存数量', dictCode: '' })
      fieldList.push({ type: 'string', value: 'userName', text: '创建人名称', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'createTime', text: '创建时间' })
      this.superFieldList = fieldList
    },
    add() {
      // this.edit(this.modelDefault)
      this.model = Object.assign({}, this.modelDefault)
      this.editDisabled = false
      this.visible = true
    },
    edit(record) {
      getAction(this.url.queryById, { id: record.id }).then(res => {
        // console.log('res', res)
        this.model = res.result.material
        this.prod.dataSource = res.result.products
      })
      // this.model = Object.assign({}, record)
      this.editDisabled = true
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        // this.prodRef
        this.$refs.prodRef.validateTable().then(errMap => {
          if (!errMap) {
            if (valid) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''
              if (!this.model.id) {
                httpurl += this.url.add
                method = 'post'
              } else {
                httpurl += this.url.edit
                method = 'put'
              }
              // if (that.prod.dataSource.length != 0) {
              //   that.model.nextProductId = ''
              //   that.prod.dataSource.forEach(val => {
              //     that.model.nextProductId += val.id + ','
              //   })
              //   that.model.nextProductId =
              //     that.model.nextProductId.substring(that.model.nextProductId.length - 1) == ','
              //       ? that.model.nextProductId.substring(0, that.model.nextProductId.length - 1)
              //       : that.model.nextProductId
              // }
              let pm = {
                material: that.model,
                products: that.prod.dataSource
              }
              httpAction(httpurl, pm, method)
                .then(res => {
                  if (res.success) {
                    that.$message.success(res.message)
                    that.$emit('ok')
                  } else {
                    that.$message.warning(res.message)
                  }
                })
                .finally(() => {
                  that.confirmLoading = false
                })
            }
          }
        })
      })
    }
  }
}
</script>
