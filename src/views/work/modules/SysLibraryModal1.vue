<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <!-- 主表单区域 -->
      <a-form-model ref="form" :model="model" :rules="validatorRules">
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code" label="入库单号">
          <a-input style="width: 100%" placeholder="请输入，忽略将自动生成" v-model="model.code" />
        </a-form-model-item>
        <a-form-model-item
          label="入库类型"
          :rules="[{ required: true, message: '入库类型不能为空!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          prop="libraryType"
        >
          <a-select defaultValue="alipay" placeholder="请选择" v-model="model.libraryType">
            <a-select-option v-for="(item, index) in recordList" :key="index" :value="item.value"
              >{{ item.label }}
            </a-select-option>
          </a-select>
        </a-form-model-item>
        <a-form-model-item
          label="入库时间"
          :rules="[{ required: true, message: '入库类型不能为空!' }]"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          prop="libraryTime"
        >
          <a-date-picker showTime valueFormat="YYYY-MM-DD HH:mm:ss" v-model="model.libraryTime" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remarks" label="备注">
          <a-input type="textarea" placeholder="请输入备注" v-model="model.remarks" />
        </a-form-model-item>
      </a-form-model>

      <!-- 子表单区域 -->
      <a-tabs v-model="activeKey" @change="handleChangeTabs">
        <a-tab-pane tab="入库明细" key="1" :forceRender="true">
          <j-editable-table
            ref="editableTable1"
            :loading="table1.loading"
            :columns="table1.columns"
            :dataSource="table1.dataSource"
            :maxHeight="300"
            :rowNumber="true"
            :rowSelection="true"
            :actionButton="true"
          />
        </a-tab-pane>
      </a-tabs>
    </a-spin>
  </a-modal>
</template>

<script>
import JEditableTable from '@/components/jeecg/JEditableTable'
import { FormTypes, VALIDATE_NO_PASSED, getRefPromise, validateFormModelAndTables } from '@/utils/JEditableTableUtil'
import { httpAction, getAction } from '@/api/manage'
import JDate from '@/components/jeecg/JDate'
import moment from 'moment'

export default {
  name: 'SysLibraryModal',
  components: {
    JDate,
    JEditableTable
  },
  data() {
    return {
      title: '操作',
      visible: false,
      confirmLoading: false,
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 6 }
      },
      validatorRules: {},
      activeKey: '1',
      // 客户信息
      table1: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '产品编码',
            key: 'code',
            width: '24%',
            type: FormTypes.input,
            defaultValue: '',
            placeholder: '请选择',
            options: [],
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '性别',
            key: 'sex',
            width: '18%',
            type: FormTypes.select,
            options: [
              // 下拉选项
              { title: '男', value: '1' },
              { title: '女', value: '2' }
            ],
            defaultValue: '',
            placeholder: '请选择${title}'
          },
          {
            title: '身份证号',
            key: 'idcard',
            width: '24%',
            type: FormTypes.input,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [
              {
                pattern: '^\\d{6}(18|19|20)?\\d{2}(0[1-9]|1[012])(0[1-9]|[12]\\d|3[01])\\d{3}(\\d|[xX])$',
                message: '${title}格式不正确'
              }
            ]
          },
          {
            title: '手机号',
            key: 'telphone',
            width: '24%',
            type: FormTypes.input,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [
              {
                pattern: '^1(3|4|5|7|8)\\d{9}$',
                message: '${title}格式不正确'
              }
            ]
          }
        ]
      },
      url: {
        add: '/longxianda/sysLibrary/add',
        edit: 'longxianda/sysLibrary/edit',
        orderCustomerList: '/test/jeecgOrderMain/queryOrderCustomerListByMainId',
        orderTicketList: '/test/jeecgOrderMain/queryOrderTicketListByMainId'
      },
      recordList: [],
      optionsList: null
    }
  },
  created() {
    this.loadRecordList()
    this.loadOptionsList()
  },
  methods: {
    // 获取所有的editableTable实例
    getAllTable() {
      return Promise.all([getRefPromise(this, 'editableTable1'), getRefPromise(this, 'editableTable2')])
    },
    add() {
      // 默认新增一条数据
      this.getAllTable().then(editableTables => {
        editableTables[0].add()
        editableTables[1].add()
      })

      this.edit({})
    },
    edit(record) {
      this.visible = true
      this.activeKey = '1'
      this.model = Object.assign({}, record)
      // 加载子表数据
      if (this.model.id) {
        let params = { id: this.model.id }
        this.requestTableData(this.url.orderCustomerList, params, this.table1)
        this.requestTableData(this.url.orderTicketList, params, this.table2)
      }
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
          // else {
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
          }
          //  else {
          //   that.$message.warning(res.message)
          // }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    close() {
      this.visible = false
      this.getAllTable().then(editableTables => {
        editableTables[0].initialize()
        editableTables[1].initialize()
      })
      this.$emit('close')
      this.$refs.form.resetFields()
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
      this.validateFields1()
    },
    handleCancel() {
      this.close()
    },
    /** ATab 选项卡切换事件 */
    handleChangeTabs(key) {
      getRefPromise(this, `editableTable${key}`).then(editableTable => {
        editableTable.resetScrollTop()
      })
    },

    /** 触发表单验证  当前使用 */
    validateFields1() {
      const that = this
      this.model.type = 1
      // 触发表单验证
      this.$refs.form.validate(valid => {
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
      })
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
          if (e.error === VALIDATE_NO_PASSED) {
            // 如果有未通过表单验证的子表，就自动跳转到它所在的tab
            this.activeKey = e.index == null ? this.activeKey : (e.index + 1).toString()
          } else {
            console.error(e)
          }
        })
    },
    /** 整理成formData */
    classifyIntoFormData(allValues) {
      let orderMain = Object.assign(this.model, allValues.formValue)
      return {
        ...orderMain, // 展开
        jeecgOrderCustomerList: allValues.tablesValue[0].values,
        jeecgOrderTicketList: allValues.tablesValue[1].values
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

<style scoped></style>
