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
    <sys-work-order-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></sys-work-order-form>
  </j-modal>
</template>

<script>
import SysWorkOrderForm from './SysWorkOrderForm'
export default {
  name: 'SysWorkOrderModal',
  components: {
    SysWorkOrderForm
  },
  data() {
    return {
      title: '',
      width: '70%',
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
