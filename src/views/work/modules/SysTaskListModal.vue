<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <sys-task-list-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></sys-task-list-form>
  </j-modal>
</template>

<script>
import SysTaskListForm from './SysTaskListForm'

export default {
  name: 'SysTaskListModal',
  components: {
    SysTaskListForm
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
      this.visible = false
      setTimeout(() => {
        this.$emit('ok')
      }, 200)
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
