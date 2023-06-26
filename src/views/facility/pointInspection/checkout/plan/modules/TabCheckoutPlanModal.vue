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
    <tab-checkout-plan-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></tab-checkout-plan-form>
  </j-modal>
</template>

<script>
import TabCheckoutPlanForm from './TabCheckoutPlanForm'
export default {
  name: 'TabCheckoutPlanModal',
  components: {
    TabCheckoutPlanForm
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
    execute(record) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.execute(record)
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
