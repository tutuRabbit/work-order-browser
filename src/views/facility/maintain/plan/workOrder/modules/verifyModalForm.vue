<template>
  <j-modal
    title="验证"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" ref="form" :rules="rules">
        <j-vxe-table
          ref="detail"
          row-number
          row-selection
          keep-source
          :height="300"
          :loading="detailTable.loading"
          :dataSource="detailTable.dataSource"
          :columns="detailTable.columns"
          style="margin-top: 8px;"
        >
          <!-- <template v-slot:action="props">
            <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
              <a>删除</a>
            </a-popconfirm>
          </template> -->
        </j-vxe-table>
        <a-row :gutter="24">
          <a-col :span="24"><div class="title">验证</div></a-col>
          <a-col :span="8">
            <a-form-item
              label="是否通过"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
              :rules="[{ required: true, message: '请选择!' }]"
            >
              <a-radio-group v-model="checkKeepOrder.isPass">
                <a-radio :value="1">是</a-radio>
                <a-radio :value="0">否</a-radio>
              </a-radio-group>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item label="评分">
              <a-rate v-model="checkKeepOrder.score" />
            </a-form-item>
          </a-col>
          <a-col :span="18">
            <a-form-item label="评价" :label-col="labelCol" :wrapper-col="wrapperCol">
              <a-input type="textarea" v-model="checkKeepOrder.evaluate"></a-input>
            </a-form-item>
          </a-col>
        </a-row>
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
      checkKeepOrder: {
        isPass: 1,
        score: 5
      },
      labelCol: { style: { width: '150px' } },
      wrapperCol: { span: 14 },
      keepOrderIds: [],
      rules: {
        isPass: [{ required: true, message: '请选择!' }]
      },
      detailTable: {
        loading: false,
        dataSource: [],
        columns: [
          // {
          //   title: '删除',
          //   key: 'action',
          //   type: JVXETypes.slot,
          //   fixed: 'left',
          //   width: '80px',
          //   align: 'center',
          //   slotName: 'action'
          // },
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
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    editor(row) {
      console.log('验证', row)
      this.detailTable.dataSource = row

      this.keepOrderIds = row.map(item => item.id)
      // console.log('this.keepOrderIds', this.keepOrderIds)

      // this.checkKeepOrder.id = row.id
      this.checkKeepOrder.keepPlanId = row[0].keepPlanId
      this.checkKeepOrder.isPass = 1
      this.checkKeepOrder.score = 5
      this.visible = true
    },
    close() {
      this.visible = false
      this.checkKeepOrder = {}
    },
    handleOk() {
      let pm = {
        keepOrderIds: this.keepOrderIds,
        checkKeepOrder: this.checkKeepOrder
      }
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
