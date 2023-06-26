<template>
  <a-spin :spinning="confirmLoading">
    <!-- <j-form-container> -->
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <a-row>
        <a-col :span="12">
          <a-form-model-item label="单据编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code">
            <a-input v-model="model.code" :disabled="formDisabled" placeholder="请输入,忽略将自动生成"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="12">
          <a-form-model-item
            label="合同编号"
            :rules="[{ required: true, message: '请填写合同编号!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="contractCode"
          >
            <a-input v-model="model.contractCode" :disabled="formDisabled" placeholder="请填写"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="12">
          <a-form-model-item
            label="客户名称"
            :rules="[{ required: true, message: '请填写客户名称!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="customName"
          >
            <a-input v-model="model.customName" :disabled="formDisabled" placeholder="请填写"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="12">
          <a-form-model-item label="重要等级" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="importantGrade">
            <j-dict-select-tag
              v-model="model.importantGrade"
              :disabled="formDisabled"
              placeholder="请选择"
              dictCode="important_grade"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="12">
          <a-form-model-item label="订单性质" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="orderNature">
            <j-dict-select-tag
              v-model="model.orderNature"
              :disabled="formDisabled"
              placeholder="请选择"
              dictCode="order_nature"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="12">
          <a-form-model-item
            label="生产车间"
            :rules="[{ required: true, message: '请选择生产车间!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="manufacturingShop"
          >
            <j-multi-select-tag
              v-model="model.manufacturingShop"
              :disabled="formDisabled"
              placeholder="请选择"
              dictCode="manufacturing_shop"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="产品列表" :labelCol="{ span: 1 }" :wrapperCol="{ span: 22 }" prop="fields">
            <template>
              <a-button v-if="!formDisabled" class="new-btn" @click="selectSparePart" type="primary">
                <a-icon type="plus" />
                添加产品
              </a-button>
              <j-vxe-table
                ref="prodRef"
                keep-source
                :height="300"
                :maxHeight="400"
                :loading="prod.loading"
                :dataSource="prod.dataSource"
                :columns="prod.columns"
                style="margin-top: 8px"
              >
                <template v-slot:planAmount="props">
                  <a-input
                    @change="planChange(props)"
                    v-model="props.value"
                    :value="props.value"
                    placeholder="请输入"
                  ></a-input>
                </template>
                <template v-slot:productionSchedule="props">
                  <a-tooltip>
                    <div
                      v-for="(item, index) in props.value"
                      :key="index"
                      style="float:left;margin-top: 15px;text-align: left;"
                    >
                      <a-progress type="circle" :width="50" :percent="item.percentage" />
                      <span v-if="props.value.length - index > 1" class="spanC">——</span>
                      <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
                    </div>
                  </a-tooltip>
                </template>
                <template v-slot:workOrderStatus="props">
                  <span>{{
                    props.value == 0 ? '未开始' : props.value == 10 ? '执行中' : props.value == null ? '-' : '已结束'
                  }}</span>
                </template>
                <template v-slot:action="props" v-if="!formDisabled">
                  <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
                    <a>删除</a>
                  </a-popconfirm>
                </template>
              </j-vxe-table>
            </template>
          </a-form-model-item>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- </j-form-container> -->
    <!-- 选择产品 -->
    <j-modal title="选择产品" width="80%" :visible="prodVis" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :md="6" :sm="12">
              <a-form-item label="快速查询">
                <a-input v-model="prodParam.search" placeholder="输产品编码、产品名称、产品规格"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="4" :lg="5" :md="6" :sm="24">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchEquipment" icon="search">查询</a-button>
                <a-button type="primary" @click="searchEquipmentReset" icon="reload" style="margin-left: 8px"
                  >重置</a-button
                >
              </span>
            </a-col>
            <a-col :md="6" :sm="12">
              <j-super-query
                :fieldList="productFieldList"
                ref="productQueryModal"
                @handleSuperQuery="handleProductQuery"
              ></j-super-query>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <!-- <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchEquipment">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="产品编码">
                <a-input placeholder="请输入" v-model.trim="prodParam.code"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="产品名称">
                <a-input placeholder="请输入" v-model.trim="prodParam.productName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="产品规格">
                <a-input placeholder="请输入" v-model.trim="prodParam.productNorms"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchEquipment" icon="search">查询</a-button>
                <a-button type="primary" @click="searchEquipmentReset" icon="reload" style="margin-left: 8px"
                  >重置</a-button
                >
              </span>
            </a-col>
          </a-row>
        </a-form>
        <j-super-query
          :fieldList="productFieldList"
          ref="productQueryModal"
          @handleSuperQuery="handleProductQuery"
        ></j-super-query>
      </div> -->
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        style="margin-top:10px"
        rowKey="id"
        :columns="prodTable.columns"
        :dataSource="prodTable.dataSource"
        :pagination="prodTable.ipagination"
        :loading="prodTable.loading"
        :rowSelection="{ selectedRowKeys: prodSelectedRowKeys, onChange: prodOnSelectChange }"
        class="j-table-force-nowrap"
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
import { filterObj } from '@/utils/util'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'SysWorkOrderForm',
  components: {},
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
      // 选择产品
      productFieldList: [],
      productQueryParams: '',
      productQueryFlag: false,
      /** 高级查询拼接方式 */
      productQueryMatchType: 'and',
      // end
      columns: [
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'procedureName'
        },
        {
          title: '工序编号',
          align: 'center',
          dataIndex: 'procedureCode'
        },
        {
          title: '报工权限',
          align: 'center',
          dataIndex: 'outputUsers'
        },
        {
          title: '报工数配比',
          align: 'center',
          dataIndex: 'outputRate'
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount'
        },
        {
          title: '不良品项列表',
          align: 'center',
          dataIndex: 'defectsStr'
        },
        {
          title: '计划开始时间',
          align: 'center',
          dataIndex: 'planStartTime'
        },
        {
          title: '计划结束时间',
          align: 'center',
          dataIndex: 'planEndTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 3 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 20 }
      },
      confirmLoading: false,
      editDisabled: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysSaleOrder/add',
        edit: '/longxianda/sysSaleOrder/edit',
        prodList: '/longxianda/sysProduct/list', //产品定义分页
        queryById: '/longxianda/sysSaleOrder/queryById'
      },
      isAdd: 0,
      ProcedureList: [], // 工序列表
      workOrderDetails: [], // 生产任务列表
      targetObj: {},
      sourceObj: {},
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
            width: '80px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '产品编号',
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
            title: '备注',
            align: 'center',
            width: '12%',
            type: this.disabled ? FormTypes.normal : FormTypes.input,
            key: 'remark'
          },
          {
            title: '数量',
            align: 'center',
            width: '12%',
            type: this.disabled ? FormTypes.normal : JVXETypes.slot,
            min: 1,
            key: 'planAmount',
            slotName: 'planAmount'
          },
          {
            title: '产品规格',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'productNorms'
          },
          {
            title: '产品属性',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'productProperty'
          },
          {
            title: '单位',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'stockPiece'
          },
          {
            title: '工单编号',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'workOrderCode'
          },
          {
            title: '状态',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'workOrderStatus',
            type: JVXETypes.slot,
            slotName: 'workOrderStatus'
          },
          {
            title: '生产进度',
            width: '35%',
            key: 'productionSchedule',
            type: JVXETypes.slot,
            align: 'center',
            slotName: 'productionSchedule'
          },
          {
            title: '完成数',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'fineAmount'
          },
          {
            title: '不良品数',
            align: 'center',
            width: '12%',
            type: FormTypes.normal,
            key: 'rejectsAmount'
          }
        ]
      },
      prodTable: {
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
        ]
      },
      prodParam: {}
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
    this.getProductFieldList()
  },
  methods: {
    planChange(val) {
      val.row.planAmount = val.value
    },
    // 选择产品
    getProductFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'code', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productNorms', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockPiece', text: '库存单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productProperty', text: '产品属性', dictCode: '' })
      fieldList.push({ type: 'string', value: 'craftRouteName', text: '工艺路线', dictCode: '' })
      this.productFieldList = fieldList
    },
    handleProductQuery(params, matchType) {
      //高级查询方法
      if (!params) {
        this.productQueryParams = ''
        this.productQueryFlag = false
      } else {
        this.productQueryFlag = true
        this.productQueryParams = JSON.stringify(params)
        this.productQueryMatchType = matchType
      }
      this.prodLoad(1)
    },
    getProductParams() {
      //获取查询条件
      let sqp = {}
      if (this.productQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.productQueryParams)
        sqp['superQueryMatchType'] = this.productQueryMatchType
      }
      var param = Object.assign(sqp, this.productParam, this.isorter, this.filters)
      param.field = this.getProductQueryField()
      param.pageNo = this.prodTable.ipagination.current
      param.pageSize = this.prodTable.ipagination.pageSize
      return filterObj(param)
    },
    getProductQueryField() {
      //TODO 字段权限控制
      var str = 'id,'
      this.prodTable.columns.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    // end
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
      let id = props.row.id
      for (var i = 0; i < this.prod.dataSource.length; i++) {
        if (this.prod.dataSource[i].id == id) {
          this.prod.dataSource.splice(i, 1)
        }
      }
    },
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      // if (Object.keys(sorter).length > 0) {
      //   this.isorter.column = sorter.field
      //   this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      // }
      this.prodTable.ipagination = pagination
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
      // this.prod.dataSource = this.prodSelectionRows
      //再次点击选择产品不会覆盖之前的产品
      const prodRowsKeys = this.prod.dataSource.map(prod => prod.id)
      if (!this.prod.dataSource.length) this.prod.dataSource = this.prodSelectionRows
      else {
        this.prodSelectionRows.forEach(prodSelection => {
          if (!prodRowsKeys.includes(prodSelection.id)) {
            this.prod.dataSource.push(prodSelection)
          }
        })
      }
      this.handleCancel()
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
        this.prodTable.ipagination.current = 1
      }
      var params = this.getProductParams() //查询条件
      params.model = 0

      var params = Object.assign(this.prodParam)
      // params.model = 0
      // params.pageNo = this.prodTable.ipagination.current
      // params.pageSize = this.prodTable.ipagination.pageSize
      this.prodTable.loading = true
      getAction(this.url.prodList, params)
        .then(res => {
          if (res.success) {
            this.prodTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.prodTable.ipagination.total = res.result.total
            } else {
              this.prodTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.prodTable.loading = false
        })
    },
    // 自定义拖拽
    customRow(record, index) {
      return {
        props: {
          draggable: 'true'
        },
        style: {
          cursor: 'pointer'
        },
        on: {
          // 鼠标移入
          mouseenter: event => {
            // 兼容IE
            var ev = event || window.event
            ev.target.draggable = true
          },
          // 开始拖拽
          dragstart: event => {
            // 兼容IE
            var ev = event || window.event
            // 阻止冒泡
            ev.stopPropagation()
            // 得到源目标数据
            this.sourceObj = record
          },
          // 拖动元素经过的元素
          dragover: event => {
            // 兼容 IE
            var ev = event || window.event
            // 阻止默认行为
            ev.preventDefault()
          },
          // 鼠标松开
          drop: event => {
            // 兼容IE
            var ev = event || window.event
            // 阻止冒泡
            ev.stopPropagation()
            // 得到目标数据
            this.targetObj = record
            const tempData = this.workOrderDetails
            tempData[this.targetObj.weight] = this.sourceObj
            tempData[this.sourceObj.weight] = this.targetObj
            let weightList = []
            tempData.forEach((item, index) => {
              item.weight = index
              weightList.push(item)
            })
            this.workOrderDetails = weightList //更改顺序
          }
        }
      }
    },
    // end
    //点击上移
    clickUp(index) {
      this.swapArray(this.workOrderDetails, index - 1, index)
    },
    //点击下移
    clickDown(index) {
      this.swapArray(this.workOrderDetails, index, index + 1)
    },
    //数组元素互换位置
    swapArray(arr, index1, index2) {
      arr[index1] = arr.splice(index2, 1, arr[index1])[0]
      return arr
    },
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.editDisabled = false
      this.visible = true
    },
    edit(record) {
      getAction(this.url.queryById, { id: record.saleOrderId }).then(res => {
        this.model = res.result
        this.prod.dataSource = res.result.products
      })
      this.editDisabled = false
      this.visible = true
    },
    detail(record) {
      getAction(this.url.queryById, { id: record.saleOrderId }).then(res => {
        this.model = res.result
        this.prod.dataSource = res.result.products
      })
      this.editDisabled = true
      this.visible = true
    },
    // add() {
    //   this.edit(this.modelDefault)
    // },
    // edit(record) {
    //   if (record.id) {
    //     this.getInfo(record.id)
    //   }
    //   this.model = Object.assign({}, record)
    //   this.visible = true
    //   if (!this.model.id) {
    //     this.loadExpandFieldValues(1009)
    //   }
    // },
    // 编辑时,通过id获取详情
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysWorkOrder/queryById?id=' + id
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            this.model = Object.assign({}, res.result)
            this.workOrderDetails = this.model.workOrderDetails
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    submitForm() {
      const that = this
      // this.model.workOrderDetails = this.workOrderDetails
      //判断产品数量是否不为空且为大于0的数字
      const isAllPositiveNumber = this.prod.dataSource.every(prod => {
        let number = parseInt(prod.planAmount)
        return !!number && number > 0
      })
      if (!isAllPositiveNumber) return that.$message.warning('各产品数量不能为空且数量必须大于0！')
      else {
        this.prod.dataSource.forEach(prod => {
          prod.planAmount = parseInt(prod.planAmount)
        })
      }
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          let i = this.model
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'post'
          }
          // if (this.model.workOrderDetails.length <= 0) {
          //   that.$message.warning('生产任务不能为空!')
          //   that.confirmLoading = false
          //   return
          // }
          httpAction(httpurl, { ...this.model, products: this.prod.dataSource }, method)
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
      })
    }
  }
}
</script>
<style scoped>
.vxe-table--fixed-left-wrapper .vxe-table--body {
  height: 88px;
}
</style>
