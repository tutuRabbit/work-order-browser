<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <div class="table-operator">
      <!-- <a-upload
        name="fixWorkOrderImages"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        :data="{ fixWorkOrderId: this.fixWorkOrderId }"
        @change="handleImportExcel"
      >
        <a-button type="info" icon="upload">图片上传</a-button>
      </a-upload> -->
      <a-button type="info" icon="upload" @click="handleAdd">图片上传</a-button>
      <a-button @click="batchDel" type="info"> <a-icon type="delete" />删除</a-button>
    </div>
    <!-- table区域-begin -->
    <a-checkbox-group v-model="idList">
      <div class="list">
        <div v-for="(item, key) in dataSource" :key="key">
          <a-checkbox :value="item.id">{{ key }}</a-checkbox>
          <div class="item">
            <div style="float: left;width:104px;height:104px;margin-right: 10px;margin: 0 8px 8px 0;">
              <div
                style="width: 100%;height: 100%;position: relative;padding: 8px;border: 1px solid #d9d9d9;border-radius: 4px;"
              >
                <img style="width: 100%;height:100%" :src="item.picPrePath" :preview="item.id" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </a-checkbox-group>
    <div v-if="!isEmpty" style="font-size: 16px; color: #000; padding: 50px; text-align: center">
      暂无数据
    </div>
    <j-modal
      title="添加图片"
      width="50%"
      height="700"
      :visible="pictureVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="选择图片">
                <a-upload
                  list-type="picture-card"
                  :file-list="pictureFileList"
                  :beforeUpload="beforeUploadMains"
                  @change="handleChangeImg"
                  :showUploadList="{
                    showPreviewIcon: false
                  }"
                >
                  <div v-if="this.pictureFileList.length < 2">
                    <!-- 确定上传图片数量 -->
                    <a-icon type="plus" />
                    <div class="ant-upload-text">上传</div>
                  </div>
                </a-upload>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, deleteAction, uploadAction } from '@/api/manage'

export default {
  name: 'faultPictures',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      isEmpty: false,
      pictureFileList: [],
      idList: [],
      pictureVis: false,
      // end
      headers: {},
      fileList: [],
      description: '存档文档',
      url: {
        list: '/fixworkorder/fixWorkOrderImage/list',
        importExcelUrl: '/fixworkorder/fixWorkOrderImage/add',
        // queryById: '/fixworkorder/fixWorkOrderImage/queryById',
        deleteBatch: '/fixworkorder/fixWorkOrderImage/deleteBatch'
      },
      dataSource: [],
      fixWorkOrderId: ''
    }
  },
  created() {},
  beforeCreate() {
    this.disableMixinCreated = true
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handleOk() {
      if (this.fixWorkOrderId) {
        let pm = new FormData()
        if (this.pictureFileList.length > 0) {
          this.pictureFileList.forEach(v => {
            pm.append('fixWorkOrderImages', v.originFileObj)
          })
        }
        pm.append('fixWorkOrderId', this.fixWorkOrderId)
        uploadAction(this.url.importExcelUrl, pm).then(res => {
          if (res.success) {
            this.$message.success(res.message)
            let am = {
              id: this.fixWorkOrderId
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
      this.pictureVis = false
      this.pictureFileList = []
      this.fileList = []
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
    handleAdd() {
      this.pictureVis = true
    },
    batchDel: function() {
      if (this.idList.length == 0) {
        this.$message.warning('请选择一条记录！')
        return
      } else {
        // var ids = ''
        // for (var a = 0; a < this.selectedRowKeys.length; a++) {
        //   ids += this.selectedRowKeys[a] + ','
        // }
        var that = this
        let pm = new URLSearchParams()
        pm.append('ids', that.idList[0])
        this.$confirm({
          title: '确认删除',
          content: '是否删除选中数据?',
          onOk: function() {
            that.loading = true
            deleteAction(that.url.deleteBatch, pm)
              .then(res => {
                if (res.success) {
                  that.$message.success(res.message)
                  let em = {
                    id: that.fixWorkOrderId
                  }
                  that.details(em)
                  that.idList = []
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                that.loading = false
              })
          }
        })
      }
    },
    beforeUpload: function(file) {
      var fileType = file.type
      if (fileType.indexOf('jpg", "jpeg", "png",') < 0) {
        this.$message.warning('请上传图片')
        return false
      }
    },
    details(val) {
      if (val) {
        this.fixWorkOrderId = val.id
      }
      getAction(this.url.list, {
        workOrderId: this.fixWorkOrderId,
        pageNo: 1,
        pageSize: 999999
      }).then(res => {
        var that = this
        that.dataSource = res.result.records
        that.isEmpty = res.result.total != 0 ? true : false
        console.log('that.isEmpty', that.isEmpty)
        // console.log('that.dataSource', this.dataSource)
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
ul {
  list-style-type: none;
}
.list {
  display: flex;
  flex-wrap: wrap;
  flex: 1;
}
.item {
  width: 104px;
  height: 104px;
  position: relative;
  margin: 0 10px 10px 0;
  cursor: pointer;
}
.div {
  width: 100%;
  height: 100%;
  position: relative;
  padding: 5px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
}
</style>
