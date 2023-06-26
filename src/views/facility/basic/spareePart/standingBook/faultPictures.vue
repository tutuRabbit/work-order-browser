<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <div class="table-operator">
      <!-- <a-upload
        name="pictures"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        :data="{ linkedId: this.linkedId }"
        @change="handleImportExcel"
      >
        <a-button type="info" icon="upload">图片上传</a-button>
      </a-upload> -->
      <a-button type="info" icon="upload" @click="handleAdd">图片上传</a-button>
      <a-button @click="batchDel" type="info"> <a-icon type="delete" />删除</a-button>
    </div>
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
                  <div v-if="this.pictureFileList.length < 10">
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
import { getAction, uploadAction, deleteAction } from '@/api/manage'

export default {
  name: 'archivedDocumentList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      pictureFileList: [],
      idList: [],
      isEmpty: false,
      pictureVis: false,
      isEmpty: false,
      // end
      headers: {},
      fileList: [],
      description: '存档文档',
      url: {
        list: '/attachment/tabAttachment/getPartPicOrDoc?id=', //查询图片
        addPartPicBind: '/attachment/tabAttachment/addPartPicBind', //新增图片
        importExcelUrl: '/attachment/tabAttachment/addPartPicBind',
        deleteBatch: '/attachment/tabAttachment/deleteBatch'
      },
      dataSource: [],
      linkedId: ''
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
      if (this.linkedId) {
        let pm = new FormData()
        if (this.pictureFileList.length > 0) {
          this.pictureFileList.forEach(v => {
            pm.append('pictures', v.originFileObj)
          })
        }
        pm.append('linkedId', this.linkedId)
        uploadAction(this.url.importExcelUrl, pm).then(res => {
          if (res.success) {
            this.$message.success(res.message)
            let am = {
              id: this.linkedId
            }
            this.handleCancel()
            this.details(am)
          } else {
            this.$message.error(res.message)
          }
        })

        this.pictureVis = false
      } else {
        this.$message.success('请选择一条数据！')
      }
    },
    handleCancel() {
      this.pictureVis = false
      this.fileList = []
      this.pictureFileList = []
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
        var that = this
        var ids = ''
        for (var a = 0; a < this.idList.length; a++) {
          ids += this.idList[a] + ','
        }
        console.log('this.idList', this.idList)
        console.log('ids', ids)
        let pm = new URLSearchParams()
        pm.append('ids', ids)
        this.$confirm({
          title: '确认删除',
          content: '是否删除选中数据?',
          onOk: function() {
            that.loading = true
            deleteAction(that.url.deleteBatch, pm)
              .then(res => {
                if (res.success) {
                  that.$message.success(res.message)
                  that.details()
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                that.loading = false
                that.idList = []
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
        this.linkedId = val.id
      }
      getAction(this.url.list + this.linkedId).then(res => {
        if (res.success) {
          this.dataSource = res.result
          if (res.result.length != 0) {
            this.isEmpty = true
          } else {
            this.isEmpty = false
          }
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
  /* height: auto; */
  min-height: 200px;
  /* margin-bottom: 50px; */
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
