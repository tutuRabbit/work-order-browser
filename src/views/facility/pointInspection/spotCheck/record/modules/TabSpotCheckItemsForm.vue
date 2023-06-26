<template>
  <a-spin :spinning="confirmLoading">
    <!-- <j-form-container :disabled="formDisabled"> -->
    <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
      <a-row> </a-row>
    </a-form-model>
    <a-button v-if="!formDisabled" class="new-btn" @click="selectSparePart" type="primary">
      <a-icon type="plus" />
      选择设备
    </a-button>
    <!-- <a-button style="margin-left:10px" :disabled="formDisabled" class="new-btn" type="primary">
      <a-icon type="plus" />
      批设标准
    </a-button> -->
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
      style="margin-top: 8px;"
    />
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      :width="width"
      :visible="spareVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">选择设备</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="queryParam.dd"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.equipmentType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="queryParam.sysOrgCode"
                  style="width: 100%"
                  :tree-data="treeData"
                  placeholder="请选择使用部门"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
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
        :columns="equipmentTable.columns"
        :dataSource="equipmentTable.dataSource"
        :pagination="equipmentTable.ipagination"
        :loading="equipmentTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleEquipmentTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
    <!-- </j-form-container> -->
  </a-spin>
</template>

<script>
import { validateDuplicateValue } from '@/utils/util'
import { httpAction, getAction } from '@/api/manage'
import { VALIDATE_FAILED, getRefPromise, validateFormModelAndTables } from '@/components/jeecg/JVxeTable/utils/vxeUtils'
import { loadCategoryData, queryDepartTreeList } from '@/api/api'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

export default {
  name: 'TabSpotCheckItemsForm',
  components: {},
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    },
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
      editDisabled: false,
      // 选择设备
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
      equipmentTable: {
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
            title: '设备编码',
            align: 'center',
            dataIndex: 'equipmentCode'
          },
          {
            title: '设备名称',
            align: 'center',
            dataIndex: 'equipmentName'
          },
          {
            title: '规格型号',
            align: 'center',
            dataIndex: 'equipmentModel'
          },
          {
            title: '使用部门',
            align: 'center',
            dataIndex: 'depart'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      disableSubmit: false,
      selectedRowKeys: [],
      dictOptions: {},
      selectionRows: [],
      //end
      title: '操作',
      visible: false,
      confirmLoading: false,
      tabStock: {
        stockInOut: 0 //0出1入
      },
      supplierList: [],
      treeData: [],
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 8 }
      },
      // 选择设备
      loading: false,
      dataSource: [],
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '设备编号',
            key: 'equipmentCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'equipmentName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'equipmentModel',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '操作人',
            key: 'operatorId',
            width: '18%',
            type: FormTypes.select,
            options: [],
            dictCode: 'sys_user,realname,id',
            placeholder: '请选择'
          },
          {
            title: '巡检项目',
            key: 'patrolItem',
            width: '10%',
            type: FormTypes.input,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          }
        ]
      },
      // end
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/spot/tabSpotCheckItems/add',
        edit: '/spot/tabSpotCheckItems/edit',
        queryById: '/spot/tabSpotCheckItems/queryById',
        equipmentList: '/equipment/equipment/list' //设备管理分页
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
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.initDictConfig()
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    searchReset() {
      this.queryParam = {}
      this.equipmentLoad(1)
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleOk() {
      var data = []
      // this.table.dataSource = this.selectionRows
      this.selectionRows.forEach(item => {
        let pm = {
          deviceId: item.id,
          equipmentCode: item.equipmentCode,
          equipmentName: item.equipmentName,
          equipmentModel: item.equipmentModel
        }
        // data.push()
        this.table.dataSource.push(pm)
      })
      this.spareVis = false
      console.log('this.selectionRows', this.selectionRows)
      console.log('  this.table.dataSource', this.table.dataSource)
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
    close() {
      this.visible = false
      this.spareVis = false
      this.getAllTable().then(editableTables => {
        this.table.dataSource = []
      })
      this.$emit('close')
    },
    // 获取所有的editableTable实例
    getAllTable() {
      return Promise.all([getRefPromise(this, 'editableTable1'), getRefPromise(this, 'editableTable2')])
    },
    handleCancel() {
      this.close()
    },
    searchQuery() {
      this.equipmentLoad(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    selectSparePart() {
      this.equipmentLoad()
      this.spareVis = true
    },
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.equipmentTable.ipagination = pagination
      this.equipmentLoad()
    },
    equipmentLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.equipmentTable.ipagination.current = 1
      }
      var params = Object.assign(this.queryParam)
      params.model = 0
      params.pageNo = this.equipmentTable.ipagination.current
      params.pageSize = this.equipmentTable.ipagination.pageSize
      this.equipmentTable.loading = true
      getAction(this.url.equipmentList, params, this.isorter)
        .then(res => {
          if (res.success) {
            this.equipmentTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.equipmentTable.ipagination.total = res.result.total
            } else {
              this.equipmentTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.equipmentTable.loading = false
        })
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      console.log('record', record)

      this.table.dataSource = Object.assign({}, record)
      console.log(' this.table.dataSource ', this.table.dataSource)
      // this.model = Object.assign({}, record)
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      // this.$refs.form.validate(valid => {
      //   if (valid) {
      that.confirmLoading = true
      let httpurl = ''
      let method = ''
      let list = this.table.dataSource
      if (list.length != 0) {
        // if (!this.model.id) {
        httpurl += this.url.add
        method = 'post'
      } else {
        httpurl += this.url.edit
        method = 'put'
      }
      httpAction(httpurl, { list: list }, method)
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
      //   }
      // })
    }
  }
}
</script>
