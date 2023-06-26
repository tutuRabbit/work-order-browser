<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="true"
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <sys-work-reporting-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></sys-work-reporting-form>
  </j-modal>
</template>

<script>
import SysWorkReportingForm from './SysWorkReportingForm'
export default {
  name: 'SysWorkReportingModal',
  components: {
    SysWorkReportingForm
  },
  data() {
    return {
      title: '',
      width: 800,
      visible: false,
      disableSubmit: false
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
