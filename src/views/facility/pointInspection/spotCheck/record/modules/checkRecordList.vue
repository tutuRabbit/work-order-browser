<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button type="primary" @click="handleJump">跳过巡检</a-button>
    </div>
    <div class="table-operator">
      <span style="font-size:16px">【项目巡检进度】：</span>
      <a-radio-group @change="searchList" v-model="queryParam.status" button-style="solid">
        <a-radio-button style="color: #f00" value="1">异常({{ this.errorCount }})</a-radio-button>
        <a-radio-button style="color: #f00" value="4">漏检({{ this.notCheckCount }})</a-radio-button>
      </a-radio-group>
      <span style="font-size:16px">【设备跳过情况】：</span>
      <a-radio-group v-model="queryParam.isOverdue" button-style="solid" @change="searchList">
        <a-radio-button style="color: #0049ff" value="1">跳过巡检({{ this.skipCheckCount }})</a-radio-button>
        <a-radio-button style="color: #0049ff" value="0">跳过扫码({{ this.skipCheckCount }})</a-radio-button>
      </a-radio-group>
      <a-button @click="handleReset" style="margin-left:10px">重置状态</a-button>
    </div>
    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
      </a-table>
    </div>
    <!-- table区域-end -->
    <j-modal title="跳过巡检" width="600px" :visible="jumpVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="跳过原因">
                <a-select
                  v-model.trim="jumpParam.skipReason"
                  :showSearch="true"
                  optionFilterProp="label"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="0" label="保养中">保养中</a-select-option>
                  <a-select-option value="1" label="外委中">外委中</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="jumpOk">确认</a-button>
      </template>
    </j-modal>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'

export default {
  name: 'checkRecordList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '设备情况',
      jumpVisible: false,
      JumpId: '',
      errorCount: 0,
      notCheckCount: 0,
      skipCheckCount: 0,
      // 表头
      columns: [
        {
          title: '标准名称',
          align: 'center',
          dataIndex: 'patrolProjectName'
        },
        {
          title: '巡检项目进度',
          align: 'center',
          dataIndex: 'spotCheckPerson'
        },
        {
          title: '巡检开始时间',
          align: 'center',
          dataIndex: 'checkStartTime'
        },
        {
          title: '巡检结束时间',
          align: 'center',
          dataIndex: 'checkEndTime'
        },
        {
          title: '设备编号',
          align: 'center',
          dataIndex: 'equipmentCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'equipmentName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'equipmentModel'
        },
        {
          title: '使用部门',
          align: 'center'
          // dataIndex: 'remark'
        },
        // {
        //   title: '跳过扫码原因',
        //   align: 'center',
        //   dataIndex: 'remark'
        // },
        {
          title: '跳过原因',
          align: 'center',
          dataIndex: 'skipReason'
        },
        {
          title: '跳过人',
          align: 'center',
          dataIndex: 'skipOperator'
        }
      ],
      jumpParam: {},
      url: {
        list: '/checkplan/patrolInspectionHistory/queryEquipmentCondition',
        equipmentSkip: '/checkplan/patrolInspectionHistory/equipmentSkip' //跳过巡检
      }
    }
  },
  created() {},
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    handleCancel() {
      this.jumpVisible = false
      this.jumpParam = {}
    },
    jumpOk() {
      getAction(this.url.equipmentSkip, this.jumpParam).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
            this.$emit('ok')
          }
        } else {
          this.$message.warning(res.message)
        }
        this.handleCancel()
      })
    },
    handleReset() {
      this.queryParam.status = ''
      this.queryParam.isOverdue = ''
    },
    searchList() {
      this.queryParam.id = this.JumpId
      // console.log('this.queryParam', this.queryParam)
      // this.details(this.queryParam,'search')
      getAction(this.url.list, this.queryParam).then(res => {
        if (res.success) {
          this.dataSource = res.result.standardList
          this.errorCount = res.result.errorCount
          this.notCheckCount = res.result.notCheckCount
          this.skipCheckCount = res.result.skipCheckCount
          console.log('res.result', res.result)
        } else {
          this.dataSource = null
        }
      })
    },
    handleJump() {
      this.jumpParam.ids = ''
      for (var a = 0; a < this.selectionRows.length; a++) {
        this.jumpParam.ids += this.selectionRows[a].id + ','
      }
      // console.log(' this.jumpParam.ids', this.jumpParam)
      // console.log(' this.jumpParam.ids', this.jumpParam.ids)
      this.jumpVisible = true
    },
    details(val) {
      if (val) {
        this.JumpId = val.id
        getAction(this.url.list, { id: val.id }).then(res => {
          if (res.success) {
            this.dataSource = res.result.standardList
            this.errorCount = res.result.errorCount
            this.notCheckCount = res.result.notCheckCount
            this.skipCheckCount = res.result.skipCheckCount
            console.log('res.result', res.result)
          } else {
            this.dataSource = null
          }
        })
      }
    }
  }
}
</script>
<style lang="less" scoped>
.ant-radio-button-wrapper {
  margin: 0 10px;
}
.ant-card {
  margin-left: -30px;
  margin-right: -30px;
}
.ant-card-body {
  padding-top: 0;
}
</style>
