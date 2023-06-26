<template>
  <j-modal title="更多查询" :width="width" :visible="visible" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="12">
            <a-form-item label="设备编号">
              <a-input v-model="queryParam.equipmentCode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="设备名称">
              <a-input v-model="queryParam.equipmentName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="规格型号">
              <a-input v-model="queryParam.equipmentModel"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="设备类别">
              <j-category-select v-model="queryParam.workerId" pcode="B03" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="保养级别">
              <j-dict-select-tag
                type="list"
                dictCode="keep_level"
                v-model.trim="queryParam.keepLevel"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="保养负责人">
              <j-dict-select-tag
                type="list"
                v-model.trim="queryParam.keepManager"
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
import { getAction } from '@api/manage'

export default {
  name: 'advancedQuery',
  components: {},
  data() {
    return {
      title: '',
      width: 600,
      visible: false,
      queryParam: {},
      url: {
        list: '/keepOrder/tabKeepOrder/getKeepPlan'
      }
    }
  },
  methods: {
    editor() {
      console.log(222)
      this.visible = true
    },
    close() {
      this.visible = false
    },
    handleOk() {
      let pm = Object.assign({}, this.queryParam)
      pm.pageNo = 1
      pm.pageSize = 999
      getAction(this.url.list, pm).then(res => {
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
