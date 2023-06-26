<template>
  <a-spin>
    <j-modal
      title="title"
      :width="width"
      :visible="visible"
      :fullscreen="fullscreen"
      switchFullscreen
      @ok="handleOk"
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <a-form-model ref="form" :model="model">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="评分" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleReason">
              <a-rate v-model="model.troubleReason" :tooltips="desc" />
              <span class="ant-rate-text">{{ desc[value - 1] }}</span>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="维修级别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixLevel">
              <j-dict-select-tag
                type="list"
                :disabled="formDisabled"
                v-model="model.fixLevel"
                dictCode="service_category"
                placeholder="请选择维修级别"
              />
            </a-form-model-item>
          </a-col>

          <a-col :span="24"> <div class="title">维修工作量</div></a-col>
        </a-row>
      </a-form-model>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { axios } from '@/utils/request'
import { loadCategoryData, queryDepartTreeList, getUserList } from '@/api/api'
import JUpload from '@/components/jeecg/JUpload'
import { filterObj } from '@/utils/util'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

export default {
  name: 'TabTroubleFixReportForm',
  components: {
    JUpload
  },
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
      fileList: [],
      keepStatusOptions: [
        {
          label: '待维修',
          value: '0'
        },
        {
          label: '待停机维修',
          value: '1'
        },
        {
          label: '维修中',
          value: '2'
        },
        {
          label: '外委中',
          value: '5'
        },
        {
          label: '待验证',
          value: '3'
        },
        {
          label: '已完成',
          value: '4'
        }
      ],
      // 添加人员
      spareVis: false,
      editDisabled: false,
      spareTable: {
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
            title: '名称',
            align: 'center',
            dataIndex: 'realname'
          },
          {
            title: '性别',
            align: 'center',
            dataIndex: 'sex_dictText'
          },
          {
            title: '职务',
            align: 'center',
            dataIndex: 'post'
          },
          {
            title: '部门',
            align: 'center',
            dataIndex: 'orgCodeTxt'
          },
          {
            title: '执行班组',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      table: {
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
            title: '姓名',
            key: 'realname',
            width: '3%',
            type: FormTypes.normal
          },
          {
            title: '维修开始时间',
            key: 'count',
            width: '10%',
            type: FormTypes.datetime,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '维修完成时间',
            key: 'unitPrice',
            width: '10%',
            type: FormTypes.datetime,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '维修用时(小时)',
            key: 'amount',
            width: '10%',
            type: FormTypes.normal,
            validateRules: [{ required: false, message: '可修改单位为分钟数' }]
          },
          {
            title: '实际维修工时',
            key: 'unitPrice',
            width: '10%',
            type: FormTypes.input,
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '内容或步骤',
            key: 'ddd',
            width: '10%',
            type: FormTypes.input
          }
        ]
      },
      // end
      // 选择设备
      treeData: [],
      dictOptions: {},
      disableSubmit: false,
      loading: false,
      dataSource: [],
      selectedRowKeys: [],
      selectionRows: [],
      // 添加人员
      spareTableSelectedRowKeys: [],
      spareTableSelectionRows: [],
      spareColumns: [
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
          // dataIndex: 'sysOrgCode'
          dataIndex: 'depart'
        },
        {
          title: '设备类型',
          align: 'center',
          dataIndex: 'equipmentType',
          customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
        }
      ],
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
      queryParam: {},
      equipmentParam: {},
      equipmentVis: false,
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
      // end
      model: {
        fixWorkOrderWorkloads: []
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      url: {
        add: '/fixworkorder/fixWorkOrder/add',
        // edit: '/fixworkorder/fixWorkOrder/edit',
        edit: '/fixworkorder/fixWorkOrder/implement',
        equipmentList: '/equipment/equipment/list', //设备管理分页
        queryById: '/troublereport/tabTroubleFixReport/queryById'
      }
    }
  },
  computed: {
    // end
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.initDictConfig()
    getAction('/equipment/equipment/newTroubleList').then(res => {
      if (res.success) {
        this.dataSource = res.result.records || []
        console.log('  this.dataSource', this.dataSource)
      }
    })

    // 用户信息
    getUserList().then(res => {
      if (res.success) {
        this.spareTable.dataSource = res.result.records || []
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
  methods: {
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
      var params = Object.assign(this.equipmentParam)
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
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
      let sum = 0
      for (let i = 0; i < this.fileList.length; i++) {
        sum += Math.round(this.fileList[i].size / 1024 / 1024)
      }
      this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'
    },
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'

        if (sum > 30) {
          this.$message.warning('文件总大小超过30MB，请修改后重新上传!')
        }
      }
      return false
    },
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          console.log('选择设备.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    spareTableOnSelectChange(spareTableSelectedRowKeys, spareTableSelectionRows) {
      this.spareTableSelectedRowKeys = spareTableSelectedRowKeys
      this.spareTableSelectionRows = spareTableSelectionRows
    },
    spareTableHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.spareTable.ipagination = pagination
      this.loadData()
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.loadData()
    },
    selectEquipment() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    handleOk() {
      this.model.equipmentCode = this.selectionRows[0].equipmentCode
      this.model.equipmentName = this.selectionRows[0].equipmentName
      this.model.equipmentType = this.selectionRows[0].equipmentType
      this.model.equipmentModel = this.selectionRows[0].equipmentModel
      this.model.depart = this.selectionRows[0].sysOrgCode
      console.log('this.model', this.model)
      console.log('this.selectionRows', this.selectionRows)
      this.equipmentVis = false
    },
    spareHandleOk() {
      this.table.dataSource = this.spareTableSelectionRows
      this.spareVis = false
    },
    handleCancel() {
      this.equipmentVis = false
      this.spareVis = false
    },
    searchQuery() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    searchReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction('/equipment/equipment/newTroubleList', params)
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
    // end
    // 关联图片
    /** upload headers */
    uploadHeaders() {
      let { originColumn: col } = this
      let headers = {}
      if (col.token === true) {
        headers['X-Access-Token'] = this.$ls.get(ACCESS_TOKEN)
      }
      return headers
    },
    /** 上传请求地址 */
    uploadAction() {
      // if (!this.originColumn.action) {
      return window._CONFIG['domianURL'] + '/sys/common/upload'
      // } else {
      //   return this.originColumn.action
      // }
    },
    handleChangeUpload(info) {
      let { originColumn: col } = this
      let { file } = info
      let value = {
        name: file.name,
        type: file.type,
        size: file.size,
        status: file.status,
        percent: file.percent
      }
      if (file.response) {
        value['responseName'] = file.response[this.responseName]
      }
      if (file.status === 'done') {
        if (typeof file.response.success === 'boolean') {
          if (file.response.success) {
            value['path'] = file.response[this.responseName]
            this.handleChangeCommon(value)
          } else {
            value['status'] = 'error'
            value['message'] = file.response.message || '未知错误'
          }
        } else {
          // 考虑到如果设置action上传路径为非jeecg-boot后台，可能不会返回 success 属性的情况，就默认为成功
          value['path'] = file.response[this.responseName]
          this.handleChangeCommon(value)
        }
      } else if (file.status === 'error') {
        value['message'] = file.response.message || '未知错误'
      }
      this.innerFile = value
    },
    // end

    uploadFilesToServer(uploadApiUrl, fileName, files) {
      let formData = new FormData()
      formData.append(fileName, files)
      //添加请求头
      const headers = {
        'Content-Type': 'multipart/form-data'
      }
      //配置头
      const request = axios.create({
        headers: headers
      })
      //开始上传
      return request
        .post(uploadApiUrl, formData)
        .then(response => {
          return Promise.resolve(response.data)
        })
        .catch(error => {
          return Promise.reject(error)
        })
    },

    selectSparePart() {
      this.spareVis = true
    },
    delSparePart() {
      console.log('删除')
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.editDisabled = true
    },
    grade(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.editDisabled = true
    },
    submitForm() {
      const that = this
      this.model.fixWorkOrderWorkloads = this.table.dataSource
      // 触发表单验证
      this.$refs.form.validate(valid => {
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
            })
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
.ant-modal-body {
  margin-bottom: 0 !important;
}
</style>
