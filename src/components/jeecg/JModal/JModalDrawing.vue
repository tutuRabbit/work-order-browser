<template>
  <a-modal title="查看图纸" :width="800" :visible="visible" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
    <div class="div-modal">
      <div class="div-item" v-for="(item, key) in dataSource" :key="key">
        <a-col :span="20">
          <span style="font-size:16px">{{ item.name }}</span>
        </a-col>
        <a-col :span="4">
          <span class="modal-span"><a @click="previewFile(item)">预览</a></span>
          <span class="modal-span"><a @click="downFile(item)">下载</a></span>
        </a-col>
      </div>
    </div>
    <template slot="footer">
      <a-button @click="handleCancel">关闭</a-button>
    </template>
    <a-modal
      title="在线预览"
      :width="800"
      :footer="false"
      :visible="picVisible"
      @cancel="picVisible = false"
      cancelText="关闭"
    >
      <div>
        <!-- <div style="height:400px"> -->
        <img style="width: 100%;height:100%" :src="imgSrc" />
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
      </template>
    </a-modal>
    <pdf-preview-modal ref="pdfModal"></pdf-preview-modal>
  </a-modal>
</template>

<script>
import { httpAction, downFile, getAction } from '@/api/manage'
import PdfPreviewModal from '@views/jeecg/modules/PdfPreviewModal'
import { duplicateCheck } from '@/api/api'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import Vue from 'vue'

export default {
  components: {
    PdfPreviewModal
  },
  name: 'JModalDrawing',
  data() {
    return {
      title: '操作',
      visible: false,
      picVisible: false,
      imgSrc: '',
      model: {},
      dataSource: [],
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
        correlationId: '/longxianda/BaseFileController/correlationId?id=',
        edit: '/sys/sysDepartRole/edit'
      }
    }
  },
  created() {},
  methods: {
    pdfPreview(name) {
      let title = name.slice(0, -4)
      const token = Vue.ls.get(ACCESS_TOKEN)
      this.headers = { 'X-Access-Token': token }
      this.$refs.pdfModal.previewFiles(title, token)
    },
    previewFile(file) {
      if (file.extension != '.pdf') {
        this.imgSrc = file.path
        this.picVisible = true
      } else {
        window.open(file.path)
      }
    },
    downFile(file) {
      console.log('file', file)
      if (file) {
        // file.path//所下载文件的网络地址
        // file.name//下载成功后保存的文件名
        var url = file.path
        var xhr = new XMLHttpRequest()
        xhr.open('GET', url, true) // 定义请求方式
        // xhr.setRequestHeader('X-Auth-token', param);  // 添加 csrf 令牌
        xhr.responseType = 'blob' // 返回类型blob(重要)
        // 定义请求完成的处理函数，请求前也可以增加加载框/禁用下载按钮逻辑
        xhr.onload = function() {
          // 请求完成
          if (this.status === 200) {
            // 返回200
            const urlObject = window.URL || window.webkitURL || window
            const export_blob = new Blob([xhr.response])
            const save_link = document.createElementNS('http://www.w3.org/1999/xhtml', 'a')
            save_link.href = urlObject.createObjectURL(export_blob)
            save_link.download = file.name
            save_link.click()

            // const newUrl = window.URL.createObjectURL(xhr.response) // 生成一个可用的临时url
            // const a = document.createElement('a') // 生成a标签调用点击事件
            // a.setAttribute('href', newUrl)
            // a.setAttribute('target', '_blank')
            // a.setAttribute('download', file.name) // 自定义文件名
            // document.body.appendChild(a)
            // a.click()
            // document.body.removeChild(a)
          }
        }
        // 发送ajax请求
        xhr.send()
      } else {
        this.$message.warning('没有可下载的文件！')
      }
    },
    edit(code) {
      // this.model = Object.assign({}, record)
      // getAction(this.url.correlationId + '11').then(res => {
      getAction(this.url.correlationId + code).then(res => {
        this.dataSource = res.result
        // console.log('  图纸数据.dataSource', this.dataSource)
      })
      this.visible = true
    },
    close() {
      this.$emit('close')
      this.picVisible = false
      this.visible = false
      // this.$refs.form.resetFields()
    },
    handleOk() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          httpAction(this.url.edit, this.model, 'put')
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
              that.close()
            })
        } else {
          return false
        }
      })
    },
    handleCancel() {
      this.close()
    },
    validateRoleCode(rule, value, callback) {
      if (/[\u4E00-\u9FA5]/g.test(value)) {
        callback('部门角色编码不可输入汉字!')
      } else {
        var params = {
          tableName: 'sys_depart_role',
          fieldName: 'role_code',
          fieldVal: value,
          dataId: this.model.id
        }
        duplicateCheck(params).then(res => {
          if (res.success) {
            callback()
          } else {
            callback(res.message)
          }
        })
      }
    }
  }
}
</script>

<style lang="less" scoped>
.div-modal {
  height: 500px;
  border: 1px solid #faf9f9;
  padding: 0 20px;
}
.div-item {
  padding: 20px 0;
}
.modal-span {
  margin-left: 20px;
}
</style>
