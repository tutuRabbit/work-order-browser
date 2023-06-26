<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper" style="margin-bottom:10px">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <div class="table-operator">
      <!-- <a-upload
        name="troubleImages"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        :data="{ troubleId: this.troubleId }"
        @change="handleImportExcel"
      >
        <a-button type="info" icon="upload">图片上传</a-button>
      </a-upload> -->
      <a-button type="info" icon="upload" @click="handleAdd">图片上传</a-button>
      <a-button @click="batchDel" type="info"> <a-icon type="delete" />删除</a-button>
    </div>
    <!-- table区域-begin -->
    <!-- <a-checkbox-group v-model="idList" v-if="isEmpty" style="overflow-y: auto;width: 100%;">
      <ul class="list">
        <li v-for="(item, i) in dataSource" :key="i" class="item">
          <div>
            <a-checkbox  :value="item.id">{{ i }}</a-checkbox>
          </div>
          <div class="div">
            <img style="width: 100%;height:100%" :src="item.imageSrc" :preview="dataSource[0].id" />
          </div>
        </li>
      </ul>
    </a-checkbox-group> -->
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
import { getAction, uploadAction, deleteAction, postAction } from '@/api/manage'

export default {
  name: 'faultPictures',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      pictureFileList: [],
      pictureVis: false,
      // 分页
      arrayValue: !this.value ? [] : this.value.split(this.spliter),
      isEmpty: false,
      idList: [],
      // end
      headers: {},
      fileList: [],
      description: '存档文档',
      url: {
        queryById: '/troublereport/troubleImage/queryById',
        importExcelUrl: 'troublereport/troubleImage/add',
        deleteBatch: '/troublereport/troubleImage/edit'
      },
      dataSource: [],
      troubleId: ''
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
      // console.log(' this.troubleId ', this.troubleId)
      if (this.troubleId) {
        let pm = new FormData()
        if (this.pictureFileList.length > 0) {
          this.pictureFileList.forEach(v => {
            pm.append('troubleImages', v.originFileObj)
          })
        }
        pm.append('troubleId', this.troubleId)
        uploadAction(this.url.importExcelUrl, pm).then(res => {
          if (res.success) {
            this.$message.success(res.message)
            let am = {
              id: this.troubleId
            }
            this.details(am)
          } else {
            this.$message.error(res.message)
          }
        })
        this.handleCancel()
        this.pictureVis = false
      } else {
        this.$message.success('请选择一条数据！')
      }
    },
    handleCancel() {
      this.pictureVis = false
      this.pictureFileList = []
      // this.queryParam = {}
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
        // console.log('this.idList', this.idList)
        // console.log('ids', ids)
        let pm = new URLSearchParams()
        pm.append('ids', ids)
        this.$confirm({
          title: '确认删除',
          content: '是否删除选中数据?',
          onOk: function() {
            that.loading = true
            postAction(that.url.deleteBatch, pm)
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
              })
          }
        })
      }
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
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
        this.troubleId = val.id
      }
      getAction(this.url.queryById, {
        id: this.troubleId
      }).then(res => {
        var that = this
        that.dataSource = res.result
        that.isEmpty = res.result.length != 0 ? true : false
        console.log('that.isEmpty', that.isEmpty)
        console.log('that.dataSource', this.dataSource)
      })
    }
  }
}
</script>
<style lang="less" scoped>
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
