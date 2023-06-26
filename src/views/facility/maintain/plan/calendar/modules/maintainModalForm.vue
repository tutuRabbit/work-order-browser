<template>
  <j-modal
    title="转保养计划"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <!-- <a-button class="new-btn" type="primary" @click="handleTime">
        <a-icon type="plus" />
        批设保养时间
      </a-button>
      <a-button class="new-btn" type="primary">
        <a-icon type="plus" />
        批设保养人员
      </a-button> -->
      <j-vxe-table
        ref="detail"
        row-number
        row-selection
        keep-source
        :height="500"
        :loading="detailTable.loading"
        :dataSource="detailTable.dataSource"
        :columns="detailTable.columns"
        style="margin-top: 8px;"
        @selectRowChange="handleSelectRowChange"
      >
        <template v-slot:action="props">
          <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
            <a>删除</a>
          </a-popconfirm>
        </template>
      </j-vxe-table>
    </div>
  </j-modal>
</template>

<script>
import { postAction } from '@api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'maintainModalForm',
  components: {},
  data() {
    return {
      title: '',
      width: 600,
      visible: false,
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
            title: '保养开始时间',
            key: 'startTime',
            type: FormTypes.datetime
          },
          {
            title: '保养结束时间',
            key: 'endTime',
            type: FormTypes.datetime
          },
          // {
          //   title: '保养人员',
          //   key: 'keepManager',
          //   type: FormTypes.normal
          // },
          {
            title: '保养人员',
            key: 'popup',
            type: JVXETypes.popup,
            width: '180px',
            popupCode: 'demo',
            field: 'name,sex,age',
            orgFields: 'name,sex,age',
            destFields: 'popup,popup_sex,popup_age'
          },
          {
            title: '计划时间',
            key: 'planMonth',
            type: FormTypes.normal
          },
          {
            title: '设备编码',
            key: 'deviceCode',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'deviceName',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'deviceModel',
            type: FormTypes.normal
          },
          {
            title: '设备类型',
            key: 'deviceType',
            type: FormTypes.normal
          },
          {
            title: '使用部门',
            key: 'sysOrgCode',
            type: FormTypes.normal
          },
          {
            title: '保养级别',
            key: 'keepLevel',
            type: FormTypes.normal
          },
          {
            title: '保养标准',
            key: 'keepStandard',
            type: FormTypes.normal
          }
        ]
      },
      fullscreen: true
    }
  },
  methods: {
    // 当选择的行变化时触发的事件
    handleSelectRowChange(event) {
      this.selectedRows = event.selectedRows
    },
    handleTime() {
      const values = this.$refs.detail.getTableData()
      console.log('获取值:', { values })
      // console.log('this.$refs.detail.getAll', this.$refs.detail.getAll() )
      console.log('this.$refs.detail.removeSelection', this.$refs.detail.removeSelection())
      console.log('this.$refs.detail.getTableData', this.$refs.detail.getTableData())
    },
    handleDL(props) {
      // 调用删除方法
      // props.target.removeRows(props.row)
      let { rowId, target } = props
      target.removeRows(rowId)
      // console.log('this.keepOrderIds', this.keepOrderIds)
      // console.log('this.detailTable.dataSource', this.detailTable.dataSource)
    },
    editor(row) {
      console.log('验证', row)
      this.detailTable.dataSource = row
      this.keepOrderIds = row.map(item => item.id)
      console.log('this.keepOrderIds', this.keepOrderIds)

      this.visible = true
    },
    close() {
      this.visible = false
      this.detailTable.dataSource = []
    },
    handleOk() {
      let pm = {
        orders: this.$refs.detail.getTableData()
      }
      // 获取可编辑表格中的数据
      // const values = this.$refs.detail.getTableData()
      // 保养年计划转保养计划
      postAction('/keepOrder/tabKeepOrder/batchAdd', pm).then(res => {
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
