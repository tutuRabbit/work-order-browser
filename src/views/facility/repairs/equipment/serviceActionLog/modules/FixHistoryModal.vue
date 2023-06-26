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
    <fix-history-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></fix-history-form>
  </j-modal>
</template>

<script>
import FixHistoryForm from './FixHistoryForm'
export default {
  name: 'FixHistoryModal',
  components: {
    FixHistoryForm
  },
  data() {
    return {
      title: '',
      width: 800,
      visible: false,
      fullscreen: true,
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
