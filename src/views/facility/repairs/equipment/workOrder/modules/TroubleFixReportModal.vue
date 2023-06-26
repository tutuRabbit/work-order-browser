<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="fullscreen"
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
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
    execute(record) {
      this.visible = true
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
