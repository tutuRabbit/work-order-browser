<template>
  <j-modal title="变更任务" :width="width" :visible="visible" @ok="handleOk" @cancel="handleCancel" cancelText="关闭">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="24">
            <a-alert
              type="info"
              showIcon
              message="变更任务只对单词巡检任务生效，不影响巡检计划的计划时间及计划巡检人"
            />
          </a-col>
          <a-col :span="24">
            <a-form-item label="计划巡检时间">
              <a-range-picker v-model="changeData.passTime" disabled format="YYYY-MM-DD HH:mm:ss" style="width:100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="变更巡检时间">
              <a-range-picker @change="onChangeRangeDate" format="YYYY-MM-DD HH:mm:ss" style="width:100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="计划巡检人">
              <a-input disabled v-model="changeData.planExecutor"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="变更人员">
              <j-select-multi-user
                v-model="changeData.patrolInspectionExecutorIds"
                :query-config="selectUserQueryConfig"
              />
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </div>
  </j-modal>
</template>

<script>
import { putAction } from '@api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'
import moment from 'moment'

export default {
  name: 'verifyModalForm',
  components: {},
  data() {
    return {
      title: '',
      width: 700,
      visible: false,
      changeData: {},
      labelCol: { style: { width: '150px' } },
      wrapperCol: { span: 14 },
      keepOrderIds: [],
      fullscreen: true,
      selectUserQueryConfig: [{ key: 'phone', label: '电话' }]
    }
  },
  methods: {
    onChangeRangeDate(value, dateString) {
      this.changeData.startTime = dateString[0]
      this.changeData.endTime = dateString[1]
      console.log('this.changeData.lastSpotCheckEndTime', this.changeData)
    },
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
      // console.log('this.keepOrderIds', this.keepOrderIds)
    },
    editor(record) {
      console.log('row', record[0])
      let row = record[0]
      this.changeData = {
        id: row.id,
        planExecutor: row.planExecutor,
        passTime: [row.planStartTime, row.planEndTime]
      }
      console.log('this.changeData', this.changeData)
      this.visible = true
    },
    close() {
      this.visible = false
      this.changeData = {}
    },
    handleOk() {
      // let pm = {
      //   keepOrderIds: this.keepOrderIds,
      //   changeData: this.changeData
      // }
      // console.log('pm', pm)
      // 变更任务
      putAction('/checkplan/patrolInspectionTask/changeTask', this.changeData).then(res => {
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
