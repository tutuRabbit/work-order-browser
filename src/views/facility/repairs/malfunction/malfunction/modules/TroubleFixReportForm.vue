<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <!-- 设备信息 -->
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">设备信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentCode">
            <a-input :disabled="true" v-model="model.equipmentCode" placeholder="请输入设备编码"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="设备名称"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="equipmentName"
            :rules="[{ required: true, message: '请输入设备名称!' }]"
          >
            <a-input :disabled="true" v-model="model.equipmentName" placeholder="请输入设备名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentType">
            <j-category-select
              :disabled="true"
              v-model="model.equipmentType"
              pcode="B03"
              placeholder="请选择设备类型"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentModel">
            <a-input :disabled="true" v-model="model.equipmentModel" placeholder="请输入规格型号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="所在部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sysOrgCode">
            <a-tree-select
              :disabled="true"
              show-search
              tree-default-expand-all
              tree-node-filter-prop="title"
              v-model="model.sysOrgCode"
              style="width: 100%"
              :tree-data="treeData"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备位置" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentAddress">
            <a-input :disabled="true" v-model="model.equipmentAddress" placeholder="请输入设备位置"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8" style="margin-left:7%">
          <a-button v-if="!disEdit" type="primary" @click="selectEquipment">
            <a-icon type="search" /> 选择设备
          </a-button>
        </a-col>
      </a-row>
      <!-- 故障信息 -->
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">故障信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item
            label="发生时间"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="troubleTime"
            :rules="[{ required: true, message: '请选择发生时间!' }]"
          >
            <j-date
              :disabled="formDisabled"
              placeholder="请选择发生时间"
              v-model="model.troubleTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="报修人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="createBy">
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.createBy"
              placeholder="请选择报修人"
              @change="$forceUpdate()"
              dictCode="sys_user,realname,id"
            />
            <!-- <a-input :disabled="formDisabled" v-model="model.createBy" placeholder="请输入报修人"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="报修人手机" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleTel">
            <a-input
              @change="$forceUpdate()"
              :disabled="formDisabled"
              v-model="model.troubleTel"
              placeholder="请输入报修人手机"
            ></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障等级" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleLevel">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.troubleLevel"
              dictCode="failure_level"
              placeholder="请选择故障等级"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleType">
            <j-category-select
              :disabled="formDisabled"
              v-model="model.troubleType"
              pcode="B05"
              placeholder="请选择故障类别"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixGroup">
            <!-- <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.fixGroup"
              dictCode="maintain_list"
              placeholder="请选择维修组"
            /> -->
            <a-tree-select
              :disabled="formDisabled"
              v-model="model.fixGroup"
              style="width: 100%"
              :tree-data="treeData"
              show-search
              tree-default-expand-all
              tree-node-filter-prop="title"
              placeholder="请选择维修组"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="紧急程度" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="urgency">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.urgency"
              dictCode="emergency_degree"
              placeholder="请选择紧急程度"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="是否停机" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stopEquipment">
            <a-radio-group v-model="model.stopEquipment" :disabled="formDisabled">
              <a-radio :value="0">否</a-radio>
              <a-radio :value="1">是</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="故障描述"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="troubleDescription"
            :rules="[{ required: true, message: '请输入故障描述!' }]"
          >
            <a-input
              type="textarea"
              :disabled="formDisabled"
              v-model="model.troubleDescription"
              placeholder="请输入故障描述"
            ></a-input>
          </a-form-model-item> </a-col
      ></a-row>
      <!-- 关联文档 -->
      <a-row v-if="!disEdit">
        <a-col :span="24">
          <div class="title">关联文档</div>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="选择文件"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <a-upload
              name="file"
              :multiple="true"
              :fileList="fileList"
              :remove="handleRemove"
              :beforeUpload="beforeUpload"
            >
              <a-button :disabled="formDisabled" type="info" icon="upload">选择文件</a-button>
            </a-upload>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="上传提示"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <span>{{ queryParam.title }}</span>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="文档类型"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <j-dict-select-tag
              :disabled="true"
              style="width:100%"
              type="list"
              v-model="queryParam.docType"
              dictCode="doc_type"
              placeholder="请选择文档类型"
            />
          </a-form-item>
        </a-col>
      </a-row>
      <!--关联图片  -->
      <a-row v-if="!disEdit">
        <a-col :span="24">
          <div class="title">关联图片</div>
        </a-col>
        <a-col :span="24">
          <a-upload
            style="margin: 20px 0 0 80px;"
            list-type="picture-card"
            :file-list="pictureFileList"
            :beforeUpload="beforeUploadMains"
            @change="handleChangeImg"
            :showUploadList="{
              showPreviewIcon: false
            }"
          >
            <div v-if="this.pictureFileList.length < 1">
              <!-- 确定上传图片数量 -->
              <a-icon type="plus" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      width="60%"
      :visible="equipmentVis"
      @ok="handleOk"
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model="equipmentParam.search"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类型">
                <j-category-select v-model="equipmentParam.equipmentType" pcode="B03" placeholder="请选择设备类型" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model="equipmentParam.sysOrgCode"
                  style="width: 100%"
                  :tree-data="treeData"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  placeholder="请选择使用部门"
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
        :columns="spareColumns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio' }"
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
import { putAction, getAction, uploadAction } from '@/api/manage'
import { axios } from '@/utils/request'
import { loadCategoryData } from '@/api/api'
import JUpload from '@/components/jeecg/JUpload'
import { mixinDevice } from '@/utils/mixin'
import store from '@/store/'
import { filterObj } from '@/utils/util'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

export default {
  name: 'TabTroubleFixReportForm',
  mixins: [JeecgListMixin, mixinDevice],
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
      // 关联文档、图片
      disEdit: false,
      pictureFileList: [],
      queryParam: {
        docType: '3'
      },
      fileList: [],
      // end
      // 选择设备
      treeData: [],
      dictOptions: {},
      disableSubmit: false,
      loading: false,
      dataSource: [],
      selectedRowKeys: [],
      selectionRows: [],
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
      validatorRules: {
        troubleTel: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      url: {
        add: '/troublereport/tabTroubleFixReport/add',
        edit: '/troublereport/tabTroubleFixReport/edit',
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
    // getAction('/equipment/equipment/newTroubleList').then(res => {
    //   if (res.success) {
    //     this.dataSource = res.result.records || []
    //   }
    // })
  },
  methods: {
    validatePhone(rule, value, callback) {
      if (value) {
        var myreg = /^[1][3,4,5,7,8][0-9]{9}$/
        if (!myreg.test(value)) {
          callback('请输入正确的手机号')
        } else {
          callback()
        }
      } else {
        callback()
      }
    },
    // 关联文档、图片
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += sizeList[i] / 1024 / 1024
          // sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'
        if (sum > 30) {
          this.$message.warning('文件总大小超过30MB，请修改后重新上传!')
        }
      }
      return false
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
      this.queryParam.title = ''
    },
    beforeUploadMains(file, e) {
      let isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png' || file.type === 'image/jpg'
      if (!isJpgOrPng) {
        this.$message.error('格式错误，只能上传jpg、jpeg、png')
        return reject(false)
      }
      return false // 阻止upload默认上传事件
    },
    handleChangeImg({ fileList }) {
      this.pictureFileList = fileList
    },
    // end
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          // console.log('选择设备.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.equipmentLoad()
    },
    selectEquipment() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    equipmentLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = Object.assign(this.equipmentParam)
      params.model = 0
      params.pageNo = this.ipagination.current
      params.pageSize = this.ipagination.pageSize
      this.loading = true
      getAction(this.url.equipmentList, params, this.isorter)
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
    handleOk() {
      // this.table.dataSource = this.selectionRows
      console.log('this.selectionRows', this.selectionRows)
      if (this.selectionRows.length != 0) {
        this.model.equipmentCode = this.selectionRows[0].equipmentCode
        this.model.equipmentId = this.selectionRows[0].id
        this.model.equipmentName = this.selectionRows[0].equipmentName
        this.model.equipmentModel = this.selectionRows[0].equipmentModel
        this.model.equipmentType = this.selectionRows[0].equipmentType
        this.model.sysOrgCode = this.selectionRows[0].sysOrgCode
        this.model.equipmentAddress = this.selectionRows[0].equipmentAddress
      }
      this.equipmentVis = false
    },
    handleCancel() {
      this.equipmentVis = false
    },
    searchQuery() {
      // this.loadData(1)
      this.equipmentLoad(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    searchReset() {
      this.queryParam = {}
      this.equipmentParam = {}
      // this.loadData(1)
      this.equipmentLoad(1)
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      // 故障报修-分页查询
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

    add() {
      // this.edit(this.modelDefault)
      this.model = Object.assign({}, this.modelDefault)
      this.model.createBy = store.getters.userInfo.id
      this.model.troubleTel = store.getters.userInfo.phone
      this.model.stopEquipment = 0
      this.visible = true
    },
    edit(record) {
      // this.model = Object.assign({}, record)
      getAction('/troublereport/tabTroubleFixReport/queryById?id=' + record.id).then(res => {
        console.log('res', res)
        this.model = Object.assign({}, res.result)
      })
      this.disEdit = true
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          if (!this.model.id) {
            // 格式化基础数据
            let pm = new FormData()
            for (let i in this.model) {
              pm.append(i, this.model[i])
            }
            if (this.fileList.length != 0) {
              this.fileList.forEach(file => {
                pm.append('troubleDocument', file)
              })
            }
            if (this.pictureFileList.length > 0) {
              this.pictureFileList.forEach(v => {
                pm.append('troubleImages', v.originFileObj)
              })
            }
            uploadAction(this.url.add, pm)
              .then(res => {
                if (res.success) {
                  that.$message.success(res.message)
                  that.$emit('ok')
                  that.confirmLoading = true
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                that.confirmLoading = false
              })
          } else {
            // 格式化基础数据
            let pm = new FormData()
            for (let i in this.model) {
              pm.append(i, this.model[i])
            }
            if (this.fileList.length != 0) {
              this.fileList.forEach(file => {
                pm.append('troubleDocument', file)
              })
            }
            if (this.pictureFileList.length > 0) {
              this.pictureFileList.forEach(v => {
                pm.append('troubleImages', v.originFileObj)
              })
            }
            // delete this.model.planFixTime
            // delete this.model.proposedTimeEndTime
            // delete this.model.proposedTimeStartTime
            // delete this.model.updateTime
            putAction(this.url.edit, pm)
              .then(res => {
                if (res.success) {
                  that.$message.success(res.message)
                  that.$emit('ok')
                  that.confirmLoading = true
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
    }
  }
}
</script>
