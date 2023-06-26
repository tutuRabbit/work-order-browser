<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    :fullscreen="true"
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <sales-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></sales-form>
  </j-modal>
</template>

<script>
import SalesForm from './SalesForm'

export default {
  name: 'SysWorkOrderModal',
  components: {
    SalesForm
  },
  data() {
    return {
      title: '',
      width: '50%',
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
    detail(record) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.detail(record)
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
