<template>
  <j-modal title="派工" :width="width" :visible="visible" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="24">
            <a-form-item label="维修班组">
              <j-dict-select-tag
                type="list"
                v-model="queryParam.sysOrgCode"
                dictCode="maintain_list"
                placeholder="请选择维修班组"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="维修人员">
              <j-dict-select-tag
                type="list"
                v-model="queryParam.workerId"
                dictCode="sys_user,realname,id"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </div>
  </j-modal>
</template>

<script>
import TroubleFixReportForm from './TroubleFixReportForm'
import { postAction } from '@api/manage'

export default {
  name: 'TroubleFixReportModal',
  components: {
    TroubleFixReportForm
  },
  data() {
    return {
      title: '',
      width: 600,
      visible: false,
      queryParam: {}
    }
  },
  methods: {
    editor() {
      this.visible = true
    },
    close() {
      this.visible = false
    },
    handleOk() {
      let pm = {
        // queryParam.queryParam
      }
      postAction('/fixworkorder/fixWorkOrderWorkload/dispatch', list).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
          }
        } else {
          this.$message.warning(res.message)
        }
        this.visible = false
      })
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
