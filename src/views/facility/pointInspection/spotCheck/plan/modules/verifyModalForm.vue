<template>
  <j-modal
    :title="title"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    :okButtonProps="{ class: { 'jee-hidden': disSubmit } }"
    cancelText="关闭"
  >
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-steps class="steps" :current="currentTab">
          <a-step title="巡检计划基本信息填写" />
          <a-step title="选择设备点位、巡检内容" />
          <a-step title="高级设置" />
        </a-steps>
        <div class="content">
          <div v-if="currentTab === 0">巡检计划基本信息填写</div>
          <div v-if="currentTab === 1">选择设备点位、巡检内容</div>
          <div v-if="currentTab === 2">高级设置</div>
        </div>
      </a-form>
    </div>
  </j-modal>
</template>

<script>
// import TroubleFixReportForm from './TroubleFixReportForm'
import { postAction } from '@api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'verifyModalForm',
  components: {
    // TroubleFixReportForm
  },
  data() {
    return {
      title: '',
      width: 600,
      visible: false,
      currentTab: 0,
      checkKeepOrder: {},
      labelCol: { style: { width: '150px' } },
      wrapperCol: { span: 14 },
      keepOrderIds: [],
      detailTable: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '80px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '保养单号',
            key: 'keepCode',
            type: FormTypes.normal
          },
          {
            title: '设备编号',
            key: 'deviceCode',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'deviceName',
            type: FormTypes.normal
          }
          // {
          //   title: '保养信息',
          //   key: 'keepManager',
          //   type: FormTypes.normal
          // },
        ]
      },
      fullscreen: true
    }
  },
  methods: {
    // handler
    nextStep() {
      if (this.currentTab < 2) {
        this.currentTab += 1
      }
    },
    prevStep() {
      if (this.currentTab > 0) {
        this.currentTab -= 1
      }
    },
    finish() {
      this.currentTab = 0
    },
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    editor(row) {
      console.log('验证', row)
      this.detailTable.dataSource = row
      // this.keepOrderIds = row.map(item => item.id)
      console.log('this.keepOrderIds', this.keepOrderIds)

      // this.checkKeepOrder.id = row.id
      this.visible = true
    },
    close() {
      this.visible = false
      this.checkKeepOrder = {}
    },
    handleOk() {
      // this.keepOrderIds = this.detailTable.dataSource
      let pm = {
        keepOrderIds: this.keepOrderIds,
        checkKeepOrder: this.checkKeepOrder
      }
      console.log('pm', pm)
      console.log('this.detailTable.dataSource', this.detailTable.dataSource)
      // 保养工单-验证
      postAction('/realKeepOrder/tabKeepOrderReal/checkKeepOrder', pm).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
            this.$emit('ok')
          }
        } else {
          this.$message.warning(res.message)
        }
        this.visible = false
      })
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
