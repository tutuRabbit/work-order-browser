<template>
  <j-modal
    :title="title"
    :width="950"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :fullscreen="true"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form-model ref="form" :model="model" :rules="validatorRules">
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code" label="入库单号">
          <a-input style="width:50%" placeholder="请输入，忽略将自动生成" v-model="model.code" />
        </a-form-model-item>
        <a-form-model-item
          label="入库类型"
          :rules="[{ required: true, message: '入库类型不能为空!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          prop="libraryType"
        >
          <a-select
            defaultValue="alipay"
            placeholder="请选择"
            v-model="model.libraryType"
            :showSearch="true"
            style="width:50%"
            optionFilterProp="label"
          >
            <a-select-option v-for="(item, index) in recordList" :key="index" :value="item.value" :label="item.label"
              >{{ item.label }}
            </a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item
          label="入库时间"
          :rules="[{ required: true, message: '入库时间不能为空!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          prop="libraryTime"
        >
          <a-date-picker
            style="width:50%"
            :defaultValue="moment(getCurrentData(), 'YYYY-MM-DD HH:mm:ss')"
            showTime
            valueFormat="YYYY-MM-DD HH:mm:ss"
            v-model="model.libraryTime"
          />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remarks" label="备注">
          <a-input style="width:50%" type="textarea" placeholder="请输入备注" v-model="model.remarks" />
        </a-form-model-item>
      </a-form-model>

      <!-- 自定义字段区域 -->
      <a-col v-if="model.expandFieldValues" :span="24">
        <a-form-model-item
          v-for="(item, index) in model.expandFieldValues"
          :label="item.name"
          :key="index"
          :rules="[{ required: item.blRequired == '1', message: '请输入产品属性!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <!-- <a-form-model-item
          v-for="(item, index) in model.expandFieldValues"
          :label="item.name"
          :key="index"
          :rules="[{ required: item.blRequired == '1', message: '请输入产品属性!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :prop="fieldValue"
        > -->
          <a-tooltip placement="topLeft">
            <!-- 字段提示 -->
            <template v-if="item.description" slot="title">
              <span>{{ item.description }}</span>
            </template>
            <!-- 字段类型 -->
            <!-- 文本 -->
            <a-input
              v-if="item.type == '文本'"
              v-model="item.expands.fieldValue"
              :placeholder="item.tips"
              style="width: 100%"
              :defaultValue="item.defaultValue"
              @change="fieldValueChange(index)"
            />
            <!-- 大文本 -->
            <a-input
              v-if="item.type == '大文本'"
              v-model="item.expands.fieldValue"
              type="textarea"
              :placeholder="item.tips"
              style="width: 100%"
              :defaultValue="item.defaultValue"
              @change="fieldValueChange(index)"
            />
            <!-- 数字 -->
            <a-input-number
              v-if="item.type == '数字'"
              v-model="item.expands.fieldValue"
              :placeholder="item.tips"
              style="width: 100%"
              :defaultValue="item.defaultValue"
              @change="fieldValueChange(index)"
            />
            <!-- 时间 -->
            <a-date-picker
              v-if="item.type == '时间'"
              v-model="item.expands.fieldValue"
              showTime
              :placeholder="item.tips"
              valueFormat="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
              @change="fieldValueChange(index)"
            />
            <!-- 单选框 -->
            <a-radio-group
              v-if="item.type == '单选框'"
              v-model="item.expands.fieldValue"
              @change="fieldValueChange(index)"
              :placeholder="item.tips"
            >
              <a-radio v-for="(itemx, index) in JSON.parse(item.options)" :key="index" :value="itemx"
                >{{ itemx }}
              </a-radio>
            </a-radio-group>
            <!-- 复选框 -->
            <a-checkbox-group v-if="item.type == '复选框'" v-model="item.expands.fieldValue" :placeholder="item.tips">
              <a-row>
                <a-col v-for="(itemx, indexx) in JSON.parse(item.options)" :key="indexx" :span="8">
                  <a-checkbox :value="itemx">{{ itemx }}</a-checkbox>
                </a-col>
              </a-row>
            </a-checkbox-group>
          </a-tooltip>
        </a-form-model-item>
      </a-col>
      <a-col :span="24">
        <a-form-model-item label="入库明细" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fields">
          <template>
            <a-button v-if="editDisabled" class="new-btn" @click="selectSparePart" type="primary">
              <a-icon type="plus" />
              选择产品
            </a-button>
            <j-vxe-table
              ref="prodRef"
              row-number
              :rowSelection="false"
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
      <!-- 子表单区域 -->
      <!-- <a-col>
        <div style="margin-bottom: 10px; padding: 0px; font-size: 16px;color: rgb(24,144,255);">入库明细</div>
        <template>
          <div class="ant-table ant-table-empty" style="border: 1px solid #e8e8e8;">
            <div class="ant-table-container">
              <div class="ant-table-content">
                <table style="table-layout: auto;">
                  <thead class="ant-table-thead">
                    <tr>
                      <th class="ant-table-cell">产品编码</th>
                      <th class="ant-table-cell">产品名称</th>
                      <th class="ant-table-cell">产品规格</th>
                      <th class="ant-table-cell">入库数量</th>
                      <th class="ant-table-cell">客户名称</th>
                      <th class="ant-table-cell">库存单位</th>
                      <th class="ant-table-cell">最大库存</th>
                      <th class="ant-table-cell">最小库存</th>
                      <th class="ant-table-cell">安全库存</th>
                      <th class="ant-table-cell">库存数量</th>
                      <th class="ant-table-cell">操作</th>
                    </tr>
                  </thead>
                  <tbody class="ant-table-tbody" v-if="listCont.length == 0">
                    <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                      <td colspan="100" class="ant-table-cell">
                        <div class="ant-empty ant-empty-normal">
                          <div class="ant-empty-image"></div>
                          <div class="ant-empty-description">暂无数据</div>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                  <tbody class="ant-table-tbody" v-else>
                    <tr v-for="(item, index) in listCont" :key="index">
                      <td class="ant-table-cell">
                        <a-select
                          show-search
                          defaultValue="alipay"
                          style="width: 190px"
                          @change="fieldsChange()"
                          v-model="item.productCode"
                        >
                          <a-select-option v-for="(itemx, index) in optionsList" :key="index" :value="itemx.code"
                            >{{ itemx.code }}
                          </a-select-option>
                        </a-select>
                      </td>
                      <td class="ant-table-cell">{{ item.productName }}</td>
                      <td class="ant-table-cell">{{ item.productNorms }}</td>
                      <td class="ant-table-cell">
                        <a-input-number
                          v-model="item.libraryNumber"
                          placeholder="请填写"
                        ></a-input-number>
                      </td>
                      <td class="ant-table-cell">
                        <a-input  v-model="item.customerName" placeholder="请填写"></a-input>
                      </td>
                      <td class="ant-table-cell">{{ item.stockPiece }}</td>
                      <td class="ant-table-cell">{{ item.maxStock }}</td>
                      <td class="ant-table-cell">{{ item.minStock }}</td>
                      <td class="ant-table-cell">{{ item.securityStock }}</td>
                      <td class="ant-table-cell">{{ item.stockNumber }}</td>
                      <td class="ant-table-cell" style="color: rgb(24,144,255);" @click="delNext(index)">
                        删除
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <div class="ant-table-footer" @click="addNext()">
              <a class="link__21LS_ link-primary__1s7Ws" style="white-space: nowrap; cursor: pointer;"
                ><span
                  role="img"
                  type="plus-circle-o"
                  class="anticon anticon-plus-circle"
                  style="margin-right: 4px; padding: 0px; font-size: 14px;"
                ></span
                >添加一行</a
              >
            </div>
          </div>
        </template>
      </a-col> -->
    </a-spin>
    <!-- 选择产品 -->
    <j-modal title="选择产品" width="60%" :visible="prodVis" @ok="handleBase" @cancel="CancelBase" cancelText="关闭">
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- <j-super-query
        :fieldList="productFieldList"
        ref="productQueryModal"
        @handleSuperQuery="handleProductQuery"
      ></j-super-query> -->
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :md="8" :sm="12">
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
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        style="margin-top:10px"
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
  </j-modal>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import moment from 'moment'
import { filterObj } from '@/utils/util'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'SysLibraryModal',
  data() {
    return {
      // 选择产品
      productFieldList: [],
      productQueryParams: '',
      productQueryFlag: false,
      /** 高级查询拼接方式 */
      productQueryMatchType: 'and',
      // end
      title: '操作',
      visible: false,
      formDisabled: false,
      editDisabled: false,
      prodParam: {},
      prodVis: false,
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
            title: '材质',
            align: 'center',
            dataIndex: 'material'
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
      prod: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '80px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '产品编码',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'productCode'
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
            title: '入库数量',
            align: 'center',
            type: FormTypes.inputNumber,
            width: '12%',
            validateRules: [{ required: true, message: '${title}不能为空' }],
            placeholder: '请输入${title}',
            key: 'libraryNumber',
            defaultValue: 0.0
          },
          {
            title: '库位',
            align: 'center',
            type: FormTypes.select,
            options: [],
            dictCode: 'storage_location',
            validateRules: [{ required: true, message: '${title}不能为空' }],
            placeholder: '请选择${title}',
            width: '12%',
            key: 'storageLocation'
          },
          {
            title: '客户名称',
            align: 'center',
            type: FormTypes.input,
            validateRules: [{ required: true, message: '${title}不能为空' }],
            placeholder: '请输入${title}',
            width: '12%',
            key: 'customerName'
          },
          {
            title: '材质',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'material'
          },
          {
            title: '库存单位',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'stockPiece'
          },
          {
            title: '最大库存',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'maxStock'
          },
          {
            title: '最小库存',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'minStock'
          },
          {
            title: '安全库存',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'securityStock'
          },
          {
            title: '库存数量',
            align: 'center',
            type: FormTypes.normal,
            width: '12%',
            key: 'stockNumber'
          }
        ]
      },
      model: { expandFieldValues: null },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 3 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 20 }
      },

      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysLibrary/add',
        prodList: '/longxianda/sysProduct/list', //产品定义分页
        edit: 'longxianda/sysLibrary/edit'
      },
      recordList: null,
      prodSelectedRowKeys: [],
      prodSelectionRows: [],
      listCont: [],
      optionsList: []
    }
  },
  created() {
    this.getProductFieldList()
    this.loadRecordList()
  },
  methods: {
    moment,
    getCurrentData() {
      this.$set(this.model, 'libraryTime', moment(new Date()).format('YYYY-MM-DD HH:mm:ss'))
      return new Date().toLocaleString()
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
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.prodTable.ipagination = pagination
      this.prodLoad()
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
    getProductParams() {
      //获取查询条件
      let sqp = {}
      if (this.productQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.productQueryParams)
        sqp['superQueryMatchType'] = this.productQueryMatchType
      }
      var param = Object.assign(sqp, this.isorter, this.filters)
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
      params.search = this.prodParam.search

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
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
      let id = props.row.id
      for (var i = 0; i < this.prod.dataSource.length; i++) {
        if (this.prod.dataSource[i].id == id) {
          this.prod.dataSource.splice(i, 1)
        }
      }
      console.log(' this.prod.dataSource', this.prod.dataSource)
    },
    // 自定义字段（赋值监听）
    fieldValueChange(i) {
      this.model.expandFieldValues.map((item, index) => {
        if (i == index) {
          item.expands.fieldName = item.fieldName
          item.expands.ids = item.id
          if (item.type == '数字') {
            item.expands.fieldValue += ''
          }
          if (item.type == '复选框') {
            item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
          }
        }
      })
    },
    add() {
      //初始化默认值
      this.edit({})
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      if (!this.model.id) {
        this.loadExpandFieldValues(1006)
        this.editDisabled = true
      } else {
        this.getInfo(this.model.id)
        this.editDisabled = false
      }
    },
    // 如果是新增,通过字典Id获取自定义字段信息
    loadExpandFieldValues(value) {
      var that = this
      let httpurl = '/longxianda/sysCustom/list?targetType=' + value
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.model.expandFieldValues = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 编辑时，获取详情
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysLibrary/queryById?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          //debugger
          if (res.code == 200) {
            that.model = res.result
            // this.listCont = that.model.libraryDetaileds
            this.prod.dataSource = that.model.libraryDetaileds
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    loadRecordList() {
      var that = this
      let httpurl = '/sys/api/queryEnableDictItemsByCode?code=warehousing_type'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.length > 0) {
            that.recordList = res
          }
          //  else {
          //   that.$message.warning(res.message)
          // }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取产品列表
    loadOptionsList() {
      var that = this
      let httpurl = '/longxianda/sysProduct/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.optionsList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    fieldsChange() {
      let that = this
      this.listCont.map((item, index) => {
        this.optionsList.map(itemx => {
          if (itemx.code == item.code) {
            that.listCont[index] = itemx
            that.listCont[index].productId = itemx.id
            that.listCont[index].productName = itemx.productName
            that.listCont[index].lastProductSpec = itemx.productNorms
            that.listCont[index].lastProductUnitName = itemx.stockPiece
            that.listCont[index].lastProductUnitId = itemx.stockPieceId
            that.listCont[index].productCode = itemx.code
            that.listCont[index].type = '1'
          }
        })
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
      this.listCont = []
      this.prod.dataSource = []
      this.$refs.form.clearValidate()
    },
    // 添加子表项
    addNext(e) {
      var listCont = this.listCont
      listCont.push({})
      if (this.optionsList.length == 0) {
        this.loadOptionsList()
      }
    },
    // 删除子表项
    delNext(index) {
      var listCont = this.listCont
      listCont.splice(index, 1)
    },
    handleBase() {
      this.prodSelectionRows.forEach(item => {
        item.productId = item.id
        item.productCode = item.code
        this.prod.dataSource.push(item)
      })
      let ojb = {}
      this.prod.dataSource = this.prod.dataSource.reduce(function(prevArr, currentItem) {
        //利用对象的键名无法重复的特点，code是唯一区别的属性值
        ojb[currentItem.code] ? '' : (ojb[currentItem.code] = true && prevArr.push(currentItem))
        return prevArr
      }, [])
      this.CancelBase()
    },
    CancelBase() {
      this.prodVis = false
      this.prodSelectedRowKeys = []
      this.prodSelectionRows = []
    },
    handleOk() {
      const that = this
      this.model.expandFieldValues.map(item => {
        if (item.type == '复选框' && item.expands.fieldValue) {
          item.expands.fieldName = item.fieldName
          item.expands.ids = item.id
          item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
        }
      })
      this.model.type = '1'
      this.model.libraryDetaileds = this.prod.dataSource
      // this.model.libraryDetaileds = this.listCont
      // code
      let num = -1
      for (let i = 0; i < this.listCont.length; i++) {
        if (this.listCont[i].code == null) {
          num = i
          this.listCont.splice(num, this.listCont.length)
        }
      }
      console.log('this.listCont', this.listCont)
      // 触发表单验证
      this.$refs.form.validate(valid => {
        this.$refs.prodRef.validateTable().then(errMap => {
          if (!errMap) {
            if (valid) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''
              let i = this.model
              //debugger
              if (!this.model.id) {
                httpurl += this.url.add
                method = 'post'
              } else {
                httpurl += this.url.edit
                method = 'put'
              }
              httpAction(httpurl, this.model, method)
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
                  that.close()
                })
            } else {
              return false
            }
          }
        })
      })
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>

<style lang="less" scoped></style>
