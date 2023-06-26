<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="fullscreen"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <tab-check-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></tab-check-form>
  </j-modal>
</template>

<script>
import TabCheckForm from './TabCheckForm'
export default {
  name: 'TabCheckModal',
  components: {
    TabCheckForm
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
