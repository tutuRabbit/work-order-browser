<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    :fullscreen="fullscreen"
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <tab-spare-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></tab-spare-form>
  </j-modal>
</template>

<script>
import TabSpareForm from './TabSpareForm'
export default {
  name: 'TabSpareModal',
  components: {
    TabSpareForm
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
