<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">添加文档</a-button>
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
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="previewFile(record)">在线预览</a>
        </span>
        <span slot="down" slot-scope="text, record">
          <a @click="downFiles(record)">下载</a>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->
    <j-modal
      title="添加文档"
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
                  <a-button v-if="this.fileList.length < 10" type="info" icon="upload">选择文件</a-button>
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
import { getAction, uploadAction } from '@/api/manage'
import { downFile } from '@/api/manage'

export default {
  name: 'archivedDocumentList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      sizeList: null,
      fileList: [],
      documentVis: false,
      description: '存档文档',
      // 表头
      columns: [
        {
          title: '查看',
          align: 'center',
          width: 100,
          dataIndex: 'action',
          fixed: 'left',
          scopedSlots: { customRender: 'action' }
        },
        {
          title: '文件名',
          align: 'center',
          width: 300,
          key: 'rowIndex',
          dataIndex: 'fileName'
        },
        {
          title: '文档类型',
          align: 'center',
          dataIndex: 'docType',
          customRender: function(t, r, index) {
            if (r.docType == '8') {
              return '保养工单'
            } else {
              return '其他类型'
            }
          }
        },
        {
          title: '上传时间',
          align: 'center',
          width: 300,
          dataIndex: 'createTime'
        },
        {
          title: '上传者',
          align: 'center',
          dataIndex: 'createBy'
        },
        {
          title: '文件类型',
          align: 'center',
          dataIndex: 'fileSuffix'
        },
        {
          title: '文档大小',
          align: 'center',
          dataIndex: 'fileCount'
        },
        {
          title: '文件下载',
          dataIndex: 'down',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'down' }
        }
      ],
      loading: false,
      dataSource: [],
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
      queryParam: {
        docType: '8'
      },
      url: {
        list: '/attachment/tabAttachment/getPartPicOrDoc?id=', //附件-查询文档
        deleteBatch: '/attachment/tabAttachment/deleteBatch',
        add: '/attachment/tabAttachment/addPartDocBind',
        download: '/attachment/tabAttachment/download?id=' //下载
      },
      linkedId: ''
    }
  },
  created() {},
  computed: {},
  methods: {
    previewFile(file) {
      window.open(file.prePath)
    },
    downFiles(file) {
      downFile(this.url.download + file.id).then(data => {
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
          link.setAttribute('download', file.fileName)
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link) //下载完成移除元素
          window.URL.revokeObjectURL(url) //释放掉blob对象
        }
      })
    },
    // handleRemove(file) {
    //   const index = this.fileList.indexOf(file)
    //   const newFileList = this.fileList.slice()
    //   newFileList.splice(index, 1)
    //   this.fileList = newFileList
    // },
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
      // console.log('val', val)
      if (val) {
        this.linkedId = val.id
        getAction(this.url.list + val.id + '&docType=8').then(res => {
          if (res.success) {
            this.dataSource = res.result
          } else {
            this.dataSource = null
          }
        })
      }
    },
    handleAdd() {
      this.queryParam.docType = '8'
      this.documentVis = true
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
<style lang="less" scoped>
.ant-modal-body {
  margin-bottom: 100px;
}
.ant-upload-list {
  height: auto;
  overflow-y: auto;
}
</style>
