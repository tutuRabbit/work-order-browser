<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="fullscreen"
    @ok="handleOk"
    @cancel="handleCancel"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    cancelText="关闭"
  >
    <!-- Ok按钮是否隐藏 -->
    <!-- :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }" -->

    <trouble-fix-report-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></trouble-fix-report-form>
  </j-modal>
</template>

<script>
import TroubleFixReportForm from './TroubleFixReportForm'
export default {
  name: 'TroubleFixReportModal',
  components: {
    TroubleFixReportForm
  },
  data() {
    return {
      title: '',
      width: 800,
      visible: false,
      disableSubmit: false,
      fullscreen: true
    }
  },
  methods: {
    add() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.add()
      })
    },
    edit(record) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.edit(record)
      })
    },
    // 执行保养
    execute(record) {
      this.visible = true
      this.disableSubmit = false
      this.$nextTick(() => {
        this.$refs.realForm.execute(record)
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      this.$refs.realForm.submitForm()
    },
    submitCallback() {
      this.$emit('ok')
      this.visible = false
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
