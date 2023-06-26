<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    @ok="handleOk"
    :fullscreen="fullscreen"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <tab-spot-check-records-form
      ref="realForm"
      @ok="submitCallback"
      :disabled="disableSubmit"
    ></tab-spot-check-records-form>
  </j-modal>
</template>

<script>
import TabSpotCheckRecordsForm from './TabSpotCheckRecordsForm'
export default {
  name: 'TabSpotCheckRecordsModal',
  components: {
    TabSpotCheckRecordsForm
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
