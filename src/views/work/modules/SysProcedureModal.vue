<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="true"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <sys-procedure-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></sys-procedure-form>
  </j-modal>
</template>

<script>
import SysProcedureForm from './SysProcedureForm'
export default {
  name: 'SysProcedureModal',
  components: {
    SysProcedureForm
  },
  data() {
    return {
      title: '',
      width: 1200,
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
