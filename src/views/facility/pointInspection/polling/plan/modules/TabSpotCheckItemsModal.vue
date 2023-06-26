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
    <tab-spot-check-items-form
      ref="realForm"
      @ok="submitCallback"
      :disabled="disableSubmit"
    ></tab-spot-check-items-form>
  </j-modal>
</template>

<script>
import TabSpotCheckItemsForm from './TabSpotCheckItemsForm'
export default {
  name: 'TabSpotCheckItemsModal',
  components: {
    TabSpotCheckItemsForm
  },
  data() {
    return {
      title: '',
      width: 800,
      //
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
