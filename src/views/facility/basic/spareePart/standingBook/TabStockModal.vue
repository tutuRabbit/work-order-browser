<template>
  <j-modal
    :title="title"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <tab-stock-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></tab-stock-form>
  </j-modal>
</template>

<script>
import TabStockForm from './TabStockForm'
export default {
  name: 'TabStockModal',
  components: {
    TabStockForm
  },
  data() {
    return {
      title: '',
      width: 800,
      fullscreen: true,
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
      this.$nextTick(() => {
        this.$refs.realForm.edit(record)
      })
      this.visible = true
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
