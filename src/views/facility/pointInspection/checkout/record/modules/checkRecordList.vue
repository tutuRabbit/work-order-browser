<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleAdd(record)">上传证书</a>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->
    <j-modal
      title="上传证书"
      width="50%"
      :visible="documentVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="选择文件">
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
            <a-col :span="24" style="margin-top:50px">
              <a-form-item label="上传提示">
                <span>{{ queryParam.title }}</span>
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="文档类型">
                <j-dict-select-tag
                  type="list"
                  :disabled="true"
                  v-model="queryParam.docType"
                  dictCode="doc_type"
                  placeholder="请选择文档类型"
                />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, uploadAction, getFileAccessHttpUrl, downFile } from '@/api/manage'

export default {
  name: 'checkRecordList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '检验信息',
      // 表头
      columns: [
        {
          title: '操作',
          align: 'center',
          width: 100,
          dataIndex: 'action',
          fixed: 'left',
          scopedSlots: { customRender: 'action' }
        },
        {
          title: '计划编号',
          align: 'center',
          width: '10%',
          dataIndex: 'planCode'
        },
        {
          title: '设备编号',
          align: 'center',
          dataIndex: 'deviceCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'deviceName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'deviceModel'
        },
        {
          title: '检验结论',
          align: 'center',
          dataIndex: 'checkResult'
        },
        {
          title: '证书编号',
          align: 'center',
          dataIndex: 'certificateNumber'
        },
        {
          title: '证书',
          align: 'center',
          dataIndex: 'g'
        },
        {
          title: '费用',
          align: 'center',
          dataIndex: 'money'
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
        }
      ],
      queryParam: {},
      fileList: [],
      documentVis: false,
      url: {
        add: '/attachment/tabAttachment/addPartDocBind',
        list: '/check_plan_record/tabCheckoutPlanRecords/queryById'
      },
      linkedId: ''
    }
  },
  created() {},
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    handleAdd(record) {
      this.linkedId = record.id
      this.queryParam.docType = '10'
      this.documentVis = true
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
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
    handleOk() {
      if (this.linkedId) {
        let pm = new FormData()
        if (this.fileList.length != 0) {
          let { fileList } = this
          fileList.forEach(file => {
            pm.append('files', file)
          })
        }
        pm.append('docType', this.queryParam.docType)
        pm.append('linkedId', this.linkedId)
        uploadAction(this.url.add, pm).then(res => {
          if (res.success) {
            this.$message.success(res.message)
            let am = {
              id: this.linkedId
            }
            this.details(am)
          } else {
            this.$message.error(res.message)
          }
        })

        this.handleCancel()
      } else {
        this.$message.success('请选择一条数据！')
      }
    },
    handleCancel() {
      this.documentVis = false
      this.fileList = []
      this.queryParam = {}
    },
    details(val) {
      if (val) {
        getAction(this.url.list, { id: val.id }).then(res => {
          if (res.success) {
            this.dataSource = res.result.choosePlan
          } else {
            this.dataSource = null
          }
        })
      }
    }
  }
}
</script>
<style scoped>
.ant-card {
  margin-left: -30px;
  margin-right: -30px;
}
.ant-card-body {
  padding-top: 0;
}
</style>
