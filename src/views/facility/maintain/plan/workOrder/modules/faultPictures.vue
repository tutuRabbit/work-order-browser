<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper" style="margin-bottom:10px">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <div class="table-operator">
      <a-upload
        name="fixWorkOrderImages"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        :data="{ fixWorkOrderId: this.fixWorkOrderId }"
        @change="handleImportExcel"
      >
        <a-button v-if="this.fileList.length < 10" type="info" icon="upload">图片上传</a-button>
      </a-upload>
      <a-button @click="batchDel" type="info"> <a-icon type="delete" />删除</a-button>
    </div>
    <!-- table区域-begin -->
    <!-- <a-checkbox-group v-model="checkList" v-if="isEmpty" style="overflow-y: auto;width: 100%;">
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
    <a-checkbox-group @change="onChange" v-for="(item, key) in dataSource" :key="key">
      <div class="list">
        <div class="item">
          <a-checkbox :value="item.id">{{ key }}</a-checkbox>
          <div class="div">
            <img style="width: 100%;height:100%" :src="item.imageSrc" :preview="dataSource[0].id" />
          </div>
        </div>
      </div>
    </a-checkbox-group>
    <div v-if="!isEmpty" style="font-size: 16px; color: #000; padding: 50px; text-align: center">
      暂无数据
    </div>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, deleteAction } from '@/api/manage'

export default {
  name: 'archivedDocumentList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      // 分页
      arrayValue: !this.value ? [] : this.value.split(this.spliter),
      isEmpty: false,
      checkList: [],
      // end
      headers: {},
      fileList: [],
      description: '存档文档',
      url: {
        list: '/fixworkorder/fixWorkOrderImage/list',
        importExcelUrl: '/fixworkorder/fixWorkOrderImage/add',
        deleteBatch: '/fixworkorder/fixWorkOrderImage/deleteBatch'
      },
      dataSource: [
        // {
        //   key: 0,
        //   fileDetails: [
        //     {
        //       imgUrl: 'https://static.jeecg.com/upload/test/3a4490d5d1cd495b826e528537a47cc1.jpg'
        //     },
        //     {
        //       imgUrl: 'https://static.jeecg.com/temp/国炬软件logo_1606575029126.png'
        //     }
        //   ]
        // }
      ],
      fixWorkOrderId: ''
    }
  },
  created() {},
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    batchDel: function() {
      console.log('this.checkList', this.checkList)
      console.log('this.checkList', typeof this.checkList)
      if (this.checkList.length == 0) {
        this.$message.warning('请选择一条记录！')
        return
      } else {
        // var ids = ''
        // for (var a = 0; a < this.selectedRowKeys.length; a++) {
        //   ids += this.selectedRowKeys[a] + ','
        // }
        var that = this
        let pm = new URLSearchParams()
        pm.append('ids', that.checkList[0])
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
              })
          }
        })
      }
    },
    onChange(val) {
      this.checkList = []
      if (val.length == 1) {
        this.checkList[0] = val[0]
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
      console.log('val', val)
      if (val) {
        this.fixWorkOrderId = val.id
      }
      getAction(this.url.list, {
        fixWorkOrderId: this.fixWorkOrderId,
        pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
          this.isEmpty = this.ipagination.total = 0 ? false : true
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
