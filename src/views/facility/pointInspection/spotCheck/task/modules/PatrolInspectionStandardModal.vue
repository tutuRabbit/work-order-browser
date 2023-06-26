<template>
  <j-modal
    :title="title"
    width="60%"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <patrol-inspection-standard-form
      ref="realForm"
      @ok="submitCallback"
      :disabled="disableSubmit"
    ></patrol-inspection-standard-form>
  </j-modal>
</template>

<script>
import PatrolInspectionStandardForm from './PatrolInspectionStandardForm'
export default {
  name: 'PatrolInspectionStandardModal',
  components: {
    PatrolInspectionStandardForm
  },
  data() {
    return {
      title: '',
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
