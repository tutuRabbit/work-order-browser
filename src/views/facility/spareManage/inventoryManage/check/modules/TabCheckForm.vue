<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="tabCheck">
      <a-row>
        <a-col :span="24">
          <div class="title">盘点信息</div>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkCode">
            <a-input :disabled="formDisabled" v-model="tabCheck.checkCode" placeholder="请输入单号"></a-input>
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
              v-model="tabCheck.storeHouse"
              :showSearch="true"
              optionFilterProp="label"
              placeholder="请选择仓库"
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
          <a-form-model-item label="经办人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operator">
            <a-input :disabled="formDisabled" v-model="tabCheck.operator" placeholder="请输入经办人"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input
              type="textarea"
              :disabled="formDisabled"
              v-model="tabCheck.remark"
              placeholder="请输入备注"
            ></a-input>
          </a-form-model-item>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- 子表单区域 -->
    <div class="title">备件信息</div>
    <a-button v-if="!editDisabled" class="new-btn" @click="selectSparePart" type="primary">
      <a-icon type="plus" />
      选择备件
    </a-button>
    <a-checkbox v-if="!editDisabled" style="margin-left:20px;font-size:18px" @change="changeCheck" v-model="checkUnit"
      >换算单位</a-checkbox
    >
    <div>
      <!-- <j-vxe-table
        ref="editableTable1"
        row-number
        row-selection
        keep-source
        :height="300"
        :loading="table.loading"
        :dataSource="table.dataSource"
        :columns="tables ? columns : table.columns"
        :disabled="editDisabled"
        @valueChange="handleValueChange"
        style="margin-top: 8px;"
      >
        <template v-slot:stockManager="props">
          <a-input-number v-if="colDisabled" @change="value => changeReal(props)" :value="props.text" />
        </template>
      </j-vxe-table> -->
      <j-editable-table
        ref="editableTable1"
        keep-source
        :maxHeight="300"
        :rowNumber="true"
        :rowSelection="true"
        :actionButton="false"
        :loading="table.loading"
        :dataSource="table.dataSource"
        :columns="tables ? columns : table.columns"
        :disabled="editDisabled"
        :valueChange="handleValueChange"
        @selectRowChange="handleSelectRowChange"
        style="margin-top: 8px;"
      >
        <template v-slot:stockManager="props">
          <a-input-number v-if="colDisabled" @change="value => changeReal(props)" :value="props.text" />
        </template>
      </j-editable-table>
    </div>
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
        bordered
        rowKey="id"
        :columns="spareColumns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { getRefPromise, validateFormModelAndTables } from '@/components/jeecg/JVxeTable/utils/vxeUtils'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { httpAction, getAction } from '@/api/manage'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterObj } from '@/utils/util'

export default {
  name: 'tabCheckForm',
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
      colDisabled: true,
      tables: false,
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
      treeData: [],
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
      selectionRows: [],
      //end
      title: '操作',
      visible: false,
      confirmLoading: false,
      tabCheck: {},
      storeHouseList: [],
      supplierList: [],
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 6 }
      },
      // 选择备件
      checkUnit: false,
      loading: false,
      dataSource: [],
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '备件编号',
            key: 'spareCode',
            width: '15%',
            type: FormTypes.normal
          },
          {
            title: '备件名称',
            key: 'spareName',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'spareModel',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '分库存',
            key: 'partStockCount',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '计量单位',
            key: 'spareCountUnit',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '账面数量',
            key: 'totalStockCount',
            width: '10%',
            // 【统计列】sum = 求和、average = 平均值
            statistics: ['sum'],
            type: FormTypes.normal
          },
          {
            title: '实际数量',
            key: 'realCount',
            width: '10%',
            // 【统计列】sum = 求和、average = 平均值
            statistics: ['sum'],
            type: FormTypes.inputNumber,
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '实际数量不能为空' }]
          },
          {
            title: '换算比例',
            key: 'converScale',
            width: '10%',
            slotName: 'converScale',
            type: JVXETypes.normal
            // type: JVXETypes.hidden
          },
          {
            title: '换算单位',
            key: 'converUnit',
            width: '10%',
            slotName: 'converUnit',
            type: JVXETypes.normal
            // type: JVXETypes.hidden
          },
          {
            title: '换算数量',
            key: 'stockManager',
            width: '10%',
            type: JVXETypes.hidden
          },
          {
            title: '盘盈数量',
            key: 'profitCount',
            width: '10%',
            disabled: true,
            // 【统计列】sum = 求和、average = 平均值
            statistics: ['sum'],
            type: FormTypes.inputNumber,
            defaultValue: '',
            placeholder: '请输入${title}'
            // validateRules: [{ required: true, message: '盘盈数量不能为空' }]
          },
          {
            title: '盘亏数量',
            key: 'lossCount',
            width: '10%',
            disabled: true,
            // 【统计列】sum = 求和、average = 平均值
            statistics: ['sum'],
            type: FormTypes.inputNumber,
            defaultValue: '',
            placeholder: '请输入${title}'
            // validateRules: [{ required: true, message: '盘亏数量不能为空' }]
          },
          {
            title: '备注',
            key: 'remark',
            width: '10%',
            type: FormTypes.normal
          }
        ]
      },
      columns: [
        {
          title: '备件编号',
          key: 'spareCode',
          width: '15%',
          type: FormTypes.normal
        },
        {
          title: '备件名称',
          key: 'spareName',
          width: '8%',
          type: FormTypes.normal
        },
        {
          title: '规格型号',
          key: 'spareModel',
          width: '8%',
          type: FormTypes.normal
        },
        {
          title: '分库存',
          key: 'partStockCount',
          width: '8%',
          type: FormTypes.normal
        },
        {
          title: '计量单位',
          key: 'spareCountUnit',
          width: '8%',
          type: FormTypes.normal
        },
        {
          title: '账面数量',
          key: 'totalStockCount',
          width: '10%',
          // 【统计列】sum = 求和、average = 平均值
          statistics: ['sum'],
          type: FormTypes.normal
        },
        {
          title: '实际数量',
          key: 'realCount',
          disabled: true,
          width: '10%',
          // 【统计列】sum = 求和、average = 平均值
          statistics: ['sum'],
          type: FormTypes.inputNumber,
          placeholder: '请输入${title}',
          validateRules: [{ required: true, message: '实际数量不能为空' }]
        },
        {
          title: '换算比例',
          key: 'converScale',
          width: '10%',
          slotName: 'converScale',
          type: JVXETypes.normal
        },
        {
          title: '换算单位',
          key: 'converUnit',
          width: '10%',
          slotName: 'converUnit',
          type: JVXETypes.normal
        },
        {
          title: '换算数量',
          key: 'stockManager',
          width: '10%',
          // type: JVXETypes.hidden
          disabled: this.colDisabled,
          type: FormTypes.inputNumber
        },
        {
          title: '盘盈数量',
          key: 'profitCount',
          width: '10%',
          disabled: true,
          // 【统计列】sum = 求和、average = 平均值
          statistics: ['sum'],
          type: FormTypes.inputNumber,
          defaultValue: '',
          placeholder: '请输入${title}'
          // validateRules: [{ required: true, message: '盘盈数量不能为空' }]
        },
        {
          title: '盘亏数量',
          key: 'lossCount',
          width: '10%',
          disabled: true,
          // 【统计列】sum = 求和、average = 平均值
          statistics: ['sum'],
          type: FormTypes.inputNumber,
          defaultValue: '',
          placeholder: '请输入${title}'
          // validateRules: [{ required: true, message: '盘亏数量不能为空' }]
        },
        {
          title: '备注',
          key: 'remark',
          width: '10%',
          type: FormTypes.normal
        }
      ],
      url: {
        add: '/check/tabCheck/addCheckOrder',
        edit: '/check/tabCheck/edit'
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    this.loadData()
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.tabCheck))

    // 仓库
    getAction('/stock/tabStore/list').then(res => {
      if (res.success) {
        this.storeHouseList = res.result.records || []
      }
    })
    // 客户
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      // getAction('/equipment/tabContactCompany/list').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.supplierList = res.result.records || []
      }
    })
  },
  methods: {
    changeReal(value) {
      console.log('value', value)
    },
    changeCheck() {
      const that = this
      // that.tables = false
      console.log('that.tables', that.tables)
      console.log(111)

      if (that.checkUnit) {
        that.table.columns[9].type = 'normal'
        console.log('that.table.columns[9]', that.table.columns)
        console.log('that.table.columns[9]', that.table.columns[9])
        that.colDisabled = true
        that.tables = true
        console.log('that.tables ', that.tables)
        console.log(222)
      } else {
        that.table.columns[9].type = 'normal'
        that.colDisabled = false
        that.tables = false
        console.log('that.colDisabled ', that.tables)
        console.log(333)
      }

      that.$forceUpdate()
      that.$forceUpdate()
      that.$forceUpdate()
    },
    handleSelectRowChange(event) {
      console.log('event', event)
    },
    handleValueChange(event) {
      console.log('event', event)
      let row = event.row
      console.log('row', row)
      // console.log('row.totalStockCount', row.totalStockCount)
      // 盘盈
      if (row) {
        if (row.realCount > row.totalStockCount) {
          row.profitCount = row.realCount - row.totalStockCount
          row.lossCount = ''
          event.column.disabled = true
        } else if (row.realCount < row.totalStockCount) {
          row.lossCount = row.totalStockCount - row.realCount
          row.profitCount = ''
        } else {
          row.lossCount = 0
          row.profitCount = 0
        }
        // 盘亏
        console.log('event', event)
        this.$forceUpdate()
      }
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
      if (this.tabCheck.storeHouse) {
        this.queryParam.storeHouse = this.tabCheck.storeHouse
        this.searchQuery()
        this.spareVis = true
      } else {
        this.$message.warning('请先选择仓库！')
      }
    },
    // 获取所有的editableTable实例
    getAllTable() {
      return Promise.all([getRefPromise(this, 'editableTable1'), getRefPromise(this, 'editableTable2')])
    },

    add() {
      this.tabCheck = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      // this.tabCheck = Object.assign({}, record)
      getAction('/check/tabCheck/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.tabCheck = res.result || []
          this.editDisabled = true
        }
      })
      getAction('/check/tabCheck/queryTabCheckDetails?id=' + record.id).then(res => {
        if (res.success) {
          this.table.dataSource = res.result.records || []
        }
      })
      this.visible = true
    },
    submitForm() {
      const that = this

      // 触发表单验证
      this.$refs.form.validate(valid => {
        this.$refs.editableTable1.validateTable().then(errMap => {
          if (!errMap) {
            if (valid) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''
              if (!this.tabCheck.id) {
                httpurl += this.url.add
                method = 'post'
              } else {
                httpurl += this.url.edit
                method = 'put'
              }
              let tableList = []
              for (var i = 0; i < that.table.dataSource.length; i++) {
                tableList.push(that.table.dataSource[i])
              }
              let parm = {
                tabCheck: this.tabCheck,
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
          }
        })
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
.title {
  background-color: #f4f4f4;
  font-size: 18px;
  padding-left: 20px;
  color: #397cc0;
  font-weight: 600;
  width: 100%;
  margin: 0px 0 5px 0;
}
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
