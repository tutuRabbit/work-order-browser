<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="tabStock">
      <a-row>
        <a-col :span="24">
          <div class="title">出库信息</div>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="出库日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockTime">
            <j-date
              :disabled="formDisabled"
              v-model="tabStock.stockTime"
              style="width:100%"
              :showTime="true"
              dateFormat="YYYY-MM-DD"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="orderNum">
            <a-input :disabled="formDisabled" v-model="tabStock.orderNum" placeholder="请输入单号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="仓库"
            :rules="[{ required: true, message: '请选择仓库!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="storeHouse"
          >
            <a-select
              :disabled="formDisabled"
              v-model="tabStock.storeHouse"
              placeholder="请选择仓库"
              :showSearch="true"
              optionFilterProp="label"
            >
              <template v-for="(item, index) of storeHouseList">
                <a-select-option :value="item.id" :key="index" :label="item.storeName">{{
                  item.storeName
                }}</a-select-option>
              </template>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="出库类型"
            :labelCol="labelCol"
            :rules="[{ required: true, message: '请选择库存操作类型!' }]"
            :wrapperCol="wrapperCol"
            prop="stockType"
          >
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="tabStock.stockType"
              dictCode="out_stock_type"
              placeholder="请选择库存操作类型"
            ></j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="departId">
            <!-- <j-dict-select-tag
              type="list"
              v-model="tabStock.departId"
              dictCode="out_stock_type"
              :disabled="formDisabled"
              placeholder="请选择部门"
            ></j-dict-select-tag> -->
            <a-tree-select
              :disabled="formDisabled"
              v-model="tabStock.departId"
              style="width: 100%"
              :tree-data="treeData"
              placeholder="请选择使用部门"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="仓库管理员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockManager">
            <a-input :disabled="formDisabled" v-model="tabStock.stockManager" placeholder="请输入仓库管理员"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="经办人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operator">
            <a-input :disabled="formDisabled" v-model="tabStock.operator" placeholder="请输入经办人"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="客户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="supplier">
            <!-- <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="tabStock.supplier"
              dictCode="out_stock_type"
              placeholder="请选择供用商"
            ></j-dict-select-tag> -->
            <a-select :disabled="formDisabled" v-model="tabStock.supplier" placeholder="请选择客户">
              <template v-for="(item, index) of supplierList">
                <a-select-option :value="item.id" :key="index">{{ item.companyName }}</a-select-option>
              </template>
            </a-select>
          </a-form-model-item>
        </a-col>
        <!-- <a-col :span="8">
          <a-form-model-item label="数量总计" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="totalCount">
            <a-input-number
              :disabled="formDisabled"
              v-model="tabStock.totalCount"
              placeholder="请输入数量总计"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="金额总计" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="totalPrice">
            <a-input-number
              :disabled="formDisabled"
              v-model="tabStock.totalPrice"
              placeholder="请输入金额总计"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col> -->
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input :disabled="formDisabled" v-model="tabStock.remark" placeholder="请输入备注"></a-input>
          </a-form-model-item>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- 子表单区域 -->
    <a-button :disabled="formDisabled" class="new-btn" @click="selectSparePart" type="primary">
      <a-icon type="plus" />
      选择备件
    </a-button>
    <j-vxe-table
      ref="editableTable1"
      row-number
      row-selection
      keep-source
      :height="300"
      :loading="table.loading"
      :dataSource="table.dataSource"
      :columns="table.columns"
      :disabled="editDisabled"
      @valueChange="handleValueChange"
      style="margin-top: 8px;"
    />
    <!-- 选择备件 -->
    <j-modal
      title="选择备件"
      :width="width"
      :visible="spareVis"
      @ok="handleOk"
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">选择备件</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model="queryParam.condition"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="8">
              <a-form-item label="类型">
                <j-dict-select-tag type="list" v-model="queryParam.spareType" dictCode="spare_type"></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <!-- <a-col :span="5">
              <a-form-item label="仓库">
                <a-select v-model="queryParam.storeHouse">
                  <template v-for="(item, index) of storeHouseList">
                    <a-select-option :value="item.id" :key="index">{{ item.storeName }}</a-select-option>
                  </template>
                </a-select>
              </a-form-item>
            </a-col> -->
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        :bordered="false"
        rowKey="id"
        :columns="spareColumns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { VALIDATE_FAILED, getRefPromise, validateFormModelAndTables } from '@/components/jeecg/JVxeTable/utils/vxeUtils'
import { httpAction, getAction } from '@/api/manage'
import { FormTypes, VALIDATE_NO_PASSED } from '@/utils/JEditableTableUtil'
import { queryDepartTreeList } from '@/api/api'
import { filterObj } from '@/utils/util'

export default {
  name: 'TabStockForm',
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
      editDisabled: false,
      // 选择备件
      queryParam: {},
      width: 900,
      spareVis: false,
      allowClear: true,
      /* 分页参数 */
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
      spareColumns: [
        {
          title: '备件编号',
          align: 'center',
          dataIndex: 'spareCode'
        },
        {
          title: '备件名称',
          align: 'center',
          dataIndex: 'spareName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'spareModel'
        },
        {
          title: '类型',
          align: 'center',
          dataIndex: 'spareType'
        },
        {
          title: '库存数',
          align: 'center',
          dataIndex: 'totalStockCount'
        },
        {
          title: '分库存',
          align: 'center',
          dataIndex: 'partStockCount'
        }
      ],
      disableSubmit: false,
      selectedRowKeys: [],
      newDataSource: [],
      newArr: [],
      selectionRows: [],
      //end
      title: '操作',
      visible: false,
      confirmLoading: false,
      tabStock: {
        stockInOut: 0 //0出1入
      },
      storeHouseList: [],
      supplierList: [],
      treeData: [],
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 6 }
      },
      // 选择备件
      loading: false,
      dataSource: [],
      rowAmount: '',
      table: {
        loading: false,
        dataSource: [],
        columns: [
          // {
          //   title: '#',
          //   width: '180px',
          //   align: 'center',
          //   dataIndex: 'rowIndex',
          //   customRender: function(text, r, index) {
          //     return text !== '合计' ? parseInt(index) + 1 : text
          //   }
          // },
          {
            title: '备件编号',
            key: 'spareCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '备件名称',
            key: 'spareName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'spareModel',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '分库存',
            key: 'partStockCount',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '计量单位',
            key: 'spareCountUnit',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '数量',
            key: 'count',
            width: '10%',
            type: FormTypes.inputNumber,
            defaultValue: '',
            // 【统计列】sum = 求和、average = 平均值
            statistics: ['sum'],
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '单价',
            key: 'unitPrice',
            // key: 'sparePrice',
            width: '10%',
            type: FormTypes.inputNumber,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '金额',
            key: 'amount',
            width: '10%',
            statistics: ['sum'],
            type: FormTypes.inputNumber,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          }
        ]
      },
      url: {
        add: '/info/tabStock/addInStorageInfo',
        edit: '/info/tabStock/edit'
        // orderCustomerList: '/test/jeecgOrderMain/queryOrderCustomerListByMainId',
        // orderTicketList: '/test/jeecgOrderMain/queryOrderTicketListByMainId'
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.tabStock))
    getAction('/detail/tabStockDetail/queryStockDetails').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.dataSource = res.result.records || []
      }
    })
    // 仓库
    getAction('/stock/tabStore/list').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.storeHouseList = res.result.records || []
      }
    })
    // 客户
    // getAction('/equipment/tabContactCompany/list').then(res => {
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.supplierList = res.result.records || []
      }
    })
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  mounted() {},
  methods: {
    handleValueChange(event) {
      let row = event.row
      row.amount = row.count * row.unitPrice
      // this.rowAmount = row.amount
      // if (row.amount == this.rowAmount) {
      //   row.amount = row.count * row.unitPrice
      // } else if (row.amount == undefined || row.amount == '') {
      //   row.amount = row.count * row.unitPrice
      // }
    },
    searchQuery() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction('/detail/tabStockDetail/queryStockDetails', params)
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
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
    },
    searchReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      console.log(pagination)
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.loadData()
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    selectSparePart() {
      this.spareVis = true
    },
    // 获取所有的editableTable实例
    getAllTable() {
      return Promise.all([getRefPromise(this, 'editableTable1'), getRefPromise(this, 'editableTable2')])
    },

    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.tabStock = Object.assign({}, record)
      getAction('/info/tabStock/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.table.dataSource = res.result.detailList || []
          this.editDisabled = true
        }
      })
      this.visible = true
    },
    submitForm() {
      const that = this

      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.tabStock.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          let tableList = []
          var totalCount = 0,
            totalPrice = 0
          for (var i = 0; i < that.table.dataSource.length; i++) {
            tableList.push(that.table.dataSource[i])
          }
          that.table.dataSource.forEach(item => {
            totalCount += item.count
            totalPrice += item.amount
          })
          this.tabStock.totalCount = totalCount
          this.tabStock.totalPrice = totalPrice
          let parm = {
            tabStock: this.tabStock,
            detailList: tableList
          }
          httpAction(httpurl, parm, method)
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
    },
    close() {
      this.visible = false
      this.spareVis = false
      this.getAllTable().then(editableTables => {
        this.table.dataSource = []
      })
      this.$emit('close')
    },
    /** 查询某个tab的数据 */
    requestTableData(url, params, tab) {
      tab.loading = true
      getAction(url, params)
        .then(res => {
          tab.dataSource = res.result || []
        })
        .finally(() => {
          tab.loading = false
        })
    },
    handleOk() {
      // this.validateFields()
      this.selectionRows.forEach(item => {
        item.unitPrice = item.sparePrice
      })
      this.table.dataSource = this.selectionRows
      this.spareVis = false
      console.log('this.selectionRows', this.selectionRows)
    },
    handleCancel() {
      this.close()
    },

    /** 触发表单验证 */
    validateFields() {
      this.getAllTable()
        .then(tables => {
          /** 一次性验证主表和所有的次表 */
          return validateFormModelAndTables(this.$refs.form, this.model, tables)
        })
        .then(allValues => {
          let formData = this.classifyIntoFormData(allValues)
          // 发起请求
          return this.requestAddOrEdit(formData)
        })
        .catch(e => {
          console.error(e)
        })
    },
    /** 整理成formData */
    classifyIntoFormData(allValues) {
      let orderMain = Object.assign(this.model, allValues.formValue)
      return {
        ...orderMain, // 展开
        jeecgOrderCustomerList: allValues.tablesValue[0].tableData,
        jeecgOrderTicketList: allValues.tablesValue[1].tableData
      }
    },
    /** 发起新增或修改的请求 */
    requestAddOrEdit(formData) {
      let url = this.url.add,
        method = 'post'
      if (this.model.id) {
        url = this.url.edit
        method = 'put'
      }
      this.confirmLoading = true
      httpAction(url, formData, method)
        .then(res => {
          if (res.success) {
            this.$message.success(res.message)
            this.$emit('ok')
            this.close()
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.confirmLoading = false
        })
    }
  }
}
</script>

<style lang="less" scoped>
/** Button按钮间距 */
// .ant-btn {
//   margin-left: 30px;
//   margin-bottom: 30px;
//   float: right;
// }
.drawer-footer {
  position: absolute;
  bottom: -8px;
  width: 100%;
  border-top: 1px solid #e8e8e8;
  padding: 10px 16px;
  text-align: right;
  left: 0;
  background: #fff;
  border-radius: 0 0 2px 2px;
}
</style>
