<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">添加报修</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('故障报修')">导出</a-button>
      <a-button v-if="selectedRowKeys.length > 0" @click="batchDel" type="info"> <a-icon type="delete" />删除</a-button>
    </div>
    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 200 }"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
      </a-table>
    </div>
    <!-- table区域-end -->
    <j-modal
      title="添加报修"
      width="50%"
      :fullscreen="true"
      :visible="addVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="table-page-search-wrapper">
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
                <a-tree-select :disabled="true" v-model="model.sysOrgCode" style="width: 100%" :tree-data="treeData">
                </a-tree-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="设备位置" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentAddress">
                <a-input :disabled="true" v-model="model.equipmentAddress" placeholder="请输入设备位置"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="8" style="margin-left:7%">
              <a-button type="primary" @click="selectEquipment"> <a-icon type="search" /> 选择设备 </a-button>
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
                <a-input v-model="model.createBy" placeholder="请输入报修人"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="报修人手机" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleTel">
                <a-input v-model="model.troubleTel" placeholder="请输入报修人手机"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="故障等级" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleLevel">
                <j-dict-select-tag
                  type="list"
                  v-model="model.troubleLevel"
                  dictCode="failure_level"
                  placeholder="请选择故障等级"
                />
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="故障类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleType">
                <j-category-select v-model="model.troubleType" pcode="B05" placeholder="请选择故障类别" />
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="维修组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixGroup">
                <a-tree-select
                  v-model="model.fixGroup"
                  style="width: 100%"
                  :tree-data="treeData"
                  placeholder="请选择维修组"
                >
                </a-tree-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="紧急程度" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="urgency">
                <j-dict-select-tag
                  type="list"
                  v-model="model.urgency"
                  dictCode="emergency_degree"
                  placeholder="请选择紧急程度"
                />
              </a-form-model-item>
            </a-col>
            <a-col :span="8">
              <a-form-model-item label="是否停机" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stopEquipment">
                <a-radio-group v-model="model.stopEquipment">
                  <a-radio value="0">否</a-radio>
                  <a-radio value="1">是</a-radio>
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
                <a-input type="textarea" v-model="model.troubleDescription" placeholder="请输入故障描述"></a-input>
              </a-form-model-item> </a-col
          ></a-row>
          <!-- 关联文档 -->
          <a-row>
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
                  <a-button type="info" icon="upload">选择文件</a-button>
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
          <a-row>
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
      </div>
    </j-modal>
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      width="60%"
      :visible="equipmentVis"
      @ok="equipmentOk"
      :okButtonProps="{ class: { 'jee-hidden': false } }"
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
        :columns="equipmentData.columns"
        :dataSource="equipmentData.dataSource"
        :pagination="equipmentData.ipagination"
        :loading="equipmentData.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio' }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { loadCategoryData } from '@/api/api'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { getAction, uploadAction, downFile } from '@/api/manage'

export default {
  name: 'malfunctionRepair',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      sizeList: null,
      description: '故障报修',
      // 表头
      columns: [
        {
          title: '详细',
          align: 'center',
          width: 100,
          dataIndex: 'name',
          fixed: 'left',
          scopedSlots: { customRender: 'action' }
        },
        {
          title: '编码',
          align: 'center',
          dataIndex: 'equipmentCode'
        },
        {
          title: '报修状态',
          align: 'center',
          dataIndex: 'status'
        },
        {
          title: '处理状态',
          align: 'center',
          dataIndex: 'dealStatus'
        },
        {
          title: '外委状态',
          align: 'center'
          // dataIndex: 'has'
        },
        {
          title: '外委维修',
          align: 'center'
          // dataIndex: 'w'
        },
        {
          title: '发生时间',
          align: 'center',
          dataIndex: 'sf'
        },
        {
          title: '创建时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '维修组',
          align: 'center',
          dataIndex: 'fixGroup'
        },
        {
          title: '故障等级',
          align: 'center',
          dataIndex: 'troubleLevel'
        },
        {
          title: '故障类别',
          align: 'center'
          // dataIndex: 'd'
        },
        {
          title: '故障描述',
          align: 'center',
          dataIndex: 'troubleDescription'
        }
      ],
      url: {
        list: '/troublereport/tabTroubleFixReport/list',
        deleteBatch: '/troublereport/tabTroubleFixReport/delete',
        equipmentExportXls: '/troublereport/tabTroubleFixReport/equipmentExportXls', //导出
        equipmentList: '/equipment/equipment/list', //设备管理分页
        add: '/fixworkorder/fixWorkOrderfault/add'
      },
      model: {
        stopEquipment: '0'
      },
      validatorRules: {
        troubleTel: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      pictureFileList: [],
      fileList: [],
      addVis: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      // 设备信息

      equipmentData: {
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
        }
      },
      dictOptions: {},
      equipmentParam: {},
      equipmentVis: false,
      id: ''
    }
  },
  created() {},
  computed: {},
  methods: {
    handleExportXls(fileName) {
      if (!fileName || typeof fileName != 'string') {
        fileName = '导出文件'
      }
      let param = this.getQueryParams()
      if (this.selectedRowKeys && this.selectedRowKeys.length > 0) {
        param['selections'] = this.selectedRowKeys.join(',')
      }
      console.log('导出参数', param)
      downFile(this.url.equipmentExportXls, param).then(data => {
        if (!data) {
          this.$message.warning('文件下载失败')
          return
        }
        if (typeof window.navigator.msSaveBlob !== 'undefined') {
          window.navigator.msSaveBlob(new Blob([data], { type: 'application/vnd.ms-excel' }), fileName + '.xls')
        } else {
          let url = window.URL.createObjectURL(new Blob([data], { type: 'application/vnd.ms-excel' }))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', fileName + '.xls')
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link) //下载完成移除元素
          window.URL.revokeObjectURL(url) //释放掉blob对象
        }
      })
    },
    // 选择设备
    equipmentOk() {
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
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          // console.log('选择设备.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
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
        this.equipmentData.ipagination.current = 1
      }
      var params = Object.assign(this.equipmentParam)
      params.model = 0
      params.pageNo = this.equipmentData.ipagination.current
      params.pageSize = this.equipmentData.ipagination.pageSize
      this.loading = true
      getAction(this.url.equipmentList, params)
        .then(res => {
          if (res.success) {
            this.equipmentData.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.equipmentData.ipagination.total = res.result.total
            } else {
              this.equipmentData.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    handleChangeImg({ fileList }) {
      this.pictureFileList = fileList
    },
    beforeUploadMains(file, e) {
      let isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png' || file.type === 'image/jpg'
      if (!isJpgOrPng) {
        this.$message.error('格式错误，只能上传jpg、jpeg、png')
        return reject(false)
      }
      return false // 阻止upload默认上传事件
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
    details(val) {
      console.log('val', val)
      this.id = val.id
      getAction(this.url.list, {
        equipmentId: val.id,
        pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        } else {
          this.dataSource = null
        }
      })
    },
    handleAdd() {
      this.addVis = true
    },
    handleOk() {
      let pm = new FormData()
      if (this.fileList.length) {
        let { fileList } = this
        // console.log('fileList', fileList)
        fileList.forEach(file => {
          // console.log('file', file)
          // pm.append('fixWorkOrderfaults', file)
        })
      }
      pm.append('faultType', this.queryParam.faultType)
      pm.append('id', this.id)
      uploadAction(this.url.add, pm).then(res => {
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.error(res.message)
        }
      })

      this.addVis = false
    },
    handleCancel() {
      this.addVis = false
      this.fileList = []
      this.queryParam = {}
    },
    loadData(arg) {
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams()
      getAction(this.url.list, {
        orderId: params.mainId,
        pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        } else {
          this.dataSource = null
        }
      })
    }
  }
}
</script>
<style>
.ant-upload-list {
  height: auto;
  overflow-y: auto;
}
</style>
