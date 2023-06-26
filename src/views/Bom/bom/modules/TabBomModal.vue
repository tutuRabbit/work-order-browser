<template>
  <j-modal
    :title="title"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <tab-bom-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></tab-bom-form>
  </j-modal>
</template>

<script>
import TabBomForm from './TabBomForm'
export default {
  name: 'TabBomModal',
  components: {
    TabBomForm
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
