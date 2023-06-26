<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="巡检周期">
              <a-select
                v-model.trim="queryParam.checkCycle"
                style="width: 100%"
                placeholder="请选择"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option value="" label="全选">全选</a-select-option>
                <a-select-option value="0" label="今日待检">今日待检</a-select-option>
                <a-select-option value="1" label="小时">小时</a-select-option>
                <a-select-option value="2" label="天">天</a-select-option>
                <a-select-option value="3" label="周">周</a-select-option>
                <a-select-option value="4" label="月">月</a-select-option>
                <a-select-option value="5" label="季">季</a-select-option>
                <a-select-option value="6" label="年">年</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="巡检人员">
              <j-dict-select-tag
                v-model="queryParam.executivePlanner"
                placeholder="请选择巡检人员"
                dictCode="sys_user,realname,id"
              />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="计划所属部门">
                <a-tree-select
                  v-model.trim="queryParam.planDepart"
                  style="width: 100%"
                  :tree-data="treeData"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  placeholder="请选择"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="设备所属部门">
                <a-tree-select
                  v-model.trim="queryParam.sysOrgCode"
                  style="width: 100%"
                  :tree-data="treeData"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  placeholder="请选择"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="计划编码">
                <a-input v-model.trim="queryParam.planNo" placeholder="请输入"> </a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="计划名称">
                <a-input v-model.trim="queryParam.planName" placeholder="请输入"> </a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!-- <a-button type="primary" @click="visible = true" icon="filter" style="margin-left: 8px">更多</a-button> -->
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdds" type="primary" icon="plus">加加加</a-button> -->
      <a-button @click="handleAdd" type="primary" icon="plus">添加</a-button>
      <a-button @click="handleBlockUp(1)" type="primary">启用</a-button>
      <a-button @click="handleBlockUp(0)" type="primary">停用</a-button>
      <a-button @click="handleExecutionTime('change')" type="primary">执行时间调整</a-button>
      <a-button @click="handleExecutionTime('jump')" type="primary">设置跳过时间</a-button>
      <a-button type="primary">打印</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
    </div>
    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
        :customRow="handleTableClick"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <a-tabs defaultActiveKey="0">
      <a-tab-pane tab="巡检记录" key="0" forceRender>
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
      <a-tab-pane tab="存档文档" key="1" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
    </a-tabs>
    <trouble-fix-report-modal ref="modalForm" @ok="modalFormOk"></trouble-fix-report-modal>
    <!-- 高级查询 -->
    <j-modal title="更多查询" width="50%" :visible="visible" @cancel="visible = false" cancelText="关闭">
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="12">
              <a-form-item label="设备编号">
                <a-input v-model="queryParam.deviceCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备名称">
                <a-input v-model="queryParam.deviceName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="规格型号">
                <a-input v-model="queryParam.deviceModel"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.deviceType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="保养级别">
                <j-dict-select-tag
                  type="list"
                  dictCode="keep_level"
                  v-model.trim="queryParam.keepLevel"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="保养负责人">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="queryParam.keepManager"
                  dictCode="sys_user,realname,id"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <div style="float: left">
          <a-button :loading="loading" @click="handleReset">重置</a-button>
        </div>
        <a-button :loading="loading" @click="visible = false">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="handleOk">查询</a-button>
      </template>
    </j-modal>
    <!-- 评分 -->
    <j-modal title="评价查看" width="45%" :visible="scoreVisible" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="评分">
                <a-rate v-model="scoreParam.score" disabled />
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="评价">
                <a-input type="textarea" :disabled="true" v-model="scoreParam.evaluate"></a-input>
                <!-- <a-textarea :disabled="true" v-model="scoreParam.deviceName" :auto-size="{ minRows: 2, maxRows: 5 }" /> -->
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button :loading="loading" @click="scoreVisible = false">关闭</a-button>
      </template>
    </j-modal>
    <!-- 验证 -->
    <verify-modal-form ref="verifyModalForm"></verify-modal-form>
    <!-- 执行时间调整 -->
    <j-modal
      title="执行时间调整"
      width="45%"
      :visible="executionTimeVisible"
      cancelText="关闭"
      @cancel="close"
      @ok="okExecutionTime"
    >
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="计划巡检时间">
                <a-range-picker showTime @change="onChangeRangeDate" format="YYYY-MM-DD HH:mm:ss" />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
    <!-- 设置跳过时间 -->
    <j-modal
      title="设置跳过时间"
      width="50%"
      :visible="jumpTimeVisible"
      cancelText="关闭"
      @cancel="close"
      @ok="okJumpTime"
    >
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="跳过时段1">
                <a-range-picker @change="onChangeRangeDate1" format="YYYY-MM-DD" />
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="跳过时段2">
                <a-range-picker @change="onChangeRangeDate2" format="YYYY-MM-DD" />
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="跳过">
                <a-checkbox-group v-model="jumpTimeData.skipWeekend">
                  <a-checkbox value="0">周六</a-checkbox>
                  <a-checkbox value="1">周日</a-checkbox>
                </a-checkbox-group>
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="跳过日期">
                <a-date-picker v-model="jumpTimeData.skipDate" valueFormat="YYYY-MM-DD" style="width:100%" allowClear />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
    <!-- add -->
    <verify-modal-form ref="verifyModalForm"></verify-modal-form>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import verifyModalForm from './modules/verifyModalForm'
import { queryDepartTreeList } from '@/api/api'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import OperatingRecordList from './modules/operatingRecordList'
import ArchivedDocumentList from './modules/archivedDocumentList'
import { putAction, getAction } from '@/api/manage'

export default {
  name: 'TabKeepOrderList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    OperatingRecordList,
    verifyModalForm,
    ArchivedDocumentList
  },
  data() {
    return {
      visible: false,
      scoreVisible: false,
      executionTimeVisible: false, //执行时间调整
      jumpTimeVisible: false, //跳过时间
      description: '保养工单表管理页面',
      treeData: [],
      dictOptions: [
        {
          label: '保养中',
          value: '1'
        },
        {
          label: '待保养',
          value: '2'
        },
        {
          label: '待保养',
          value: '3'
        },
        {
          label: '保养中',
          value: '4'
        },
        {
          label: '待验证',
          value: '5'
        },
        {
          label: '已完成',
          value: '6'
        },
        {
          label: '已逾期',
          value: '7'
        }
      ],
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '计划编号',
          align: 'center',
          dataIndex: 'planNo'
        },
        {
          title: '计划状态',
          align: 'center',
          dataIndex: 'planStatus',
          customRender: function(t, r, index) {
            if (r.planStatus == '0') {
              return '启用'
            } else if (r.planStatus == '1') {
              return '停用'
            }
          }
        },
        {
          title: '计划名称',
          align: 'center',
          dataIndex: 'planName'
        },
        {
          title: '计划所属部门',
          align: 'center',
          dataIndex: 'planDepart'
        },
        {
          title: '巡检周期',
          align: 'center',
          dataIndex: 'checkCycle'
        },
        {
          title: '巡检周期时间单位',
          align: 'center',
          dataIndex: 'checkCycleTimeUnit',
          customRender: function(t, r, index) {
            if (r.checkCycleTimeUnit == 'time1') {
              return '天'
            } else if (r.checkCycleTimeUnit == 'time2') {
              return '月'
            } else if (r.checkCycleTimeUnit == 'time3') {
              return '季'
            } else {
              return '年'
            }
          }
        },
        {
          title: '计划巡检开始时间',
          align: 'center',
          dataIndex: 'startTime'
        },
        {
          title: '计划巡检结束时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '巡检人员',
          align: 'center',
          dataIndex: 'executivePlanner'
        },
        // {
        //   title: '提醒时间',
        //   align: 'center',
        //   dataIndex: 'warnTime'
        // },
        {
          title: '计划截止时间',
          align: 'center',
          dataIndex: 'cutOfTime'
        },

        {
          title: '总设备数',
          align: 'center',
          dataIndex: 'equipmentCount'
        },
        // {
        //   title: '设备范围',
        //   align: 'center',
        //   dataIndex: 'equipmentLimits'
        // },
        {
          title: '近7天未检任务次数	',
          align: 'center',
          dataIndex: 'weekExecutiveTime'
        },
        {
          title: '近7天异常任务次数',
          align: 'center',
          dataIndex: 'weekErrorTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/checkplan/patrolInspectionPlan/list',
        delete: '/checkplan/patrolInspectionPlan/delete',
        deleteBatch: '/checkplan/patrolInspectionPlan/deleteBatch',
        exportXlsUrl: '/keepOrder/tabKeepOrder/exportXls',
        startPlan: '/checkplan/patrolInspectionPlan/startPlan', //启用
        stopPlan: '/checkplan/patrolInspectionPlan/stopPlan', //停用
        adjustExecutionTime: '/checkplan/patrolInspectionPlan/adjustExecutionTime', //调整执行时间
        setSkipTime: '/checkplan/patrolInspectionPlan/setSkipTime', //设置跳过时间
        importExcelUrl: 'keepOrder/tabKeepOrder/importExcel'
      },
      executionTime: {},
      dictOptions: {},
      scoreParam: {},
      jumpTimeData: {
        skipTimeOne: '',
        skipTimeOneEnd: '',
        skipTimeTwo: '',
        skipTimeTwoEnd: '',
        skipWeekend: '',
        skipDate: ''
      },
      superFieldList: []
    }
  },
  created() {
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handleDetail: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '详情'
      this.$refs.modalForm.disableSubmit = true
      this.$refs.modalForm.disSubmit = true
    },
    close() {
      this.executionTimeVisible = false
      this.jumpTimeVisible = false
    },
    onChangeRangeDate(value, dateString) {
      this.executionTime.startTime = dateString[0]
      this.executionTime.endTime = dateString[1]
      // console.log('this.executionTime.lastSpotCheckEndTime', this.executionTime)
    },
    onChangeRangeDate1(value, dateString) {
      this.jumpTimeData.skipTimeOne = dateString[0]
      this.jumpTimeData.skipTimeOneEnd = dateString[1]
      // console.log('this.jumpTimeData.lastSpotCheckEndTime', this.jumpTimeData)
    },
    onChangeRangeDate2(value, dateString) {
      this.jumpTimeData.skipTimeTwo = dateString[0]
      this.jumpTimeData.skipTimeTwoEnd = dateString[1]
      // console.log('this.jumpTimeData.lastSpotCheckEndTime', this.jumpTimeData)
    },
    handleBlockUp(val) {
      console.log('val', val)
      console.log('this.selectionRows[0].planStatus', this.selectionRows[0].planStatus)
      if (this.selectedRowKeys.length == 1) {
        if (this.selectionRows[0].planStatus == val) {
          var id = this.selectedRowKeys.map(item => item)
          var pm = new URLSearchParams()
          pm.append('ids', id)
          if (val == 0) {
            //  启用stopPlan stopPlan
            putAction(this.url.stopPlan, pm).then(res => {
              if (res.success) {
                this.$message.success(res.message)
              } else {
                this.$message.warning(res.message)
              }
              setTimeout(() => {
                this.loadData()
              }, 500)
            })
          } else {
            // 停用
            putAction(this.url.startPlan, pm).then(res => {
              if (res.success) {
                this.$message.success(res.message)
              } else {
                this.$message.warning(res.message)
              }
              setTimeout(() => {
                this.loadData()
              }, 500)
            })
          }
        } else {
          this.$message.warning('计划状态不符,请重试！')
        }
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    handleExecutionTime(val) {
      if (this.selectedRowKeys.length == 1) {
        if (val == 'change') {
          this.executionTimeVisible = true
        } else {
          this.jumpTimeVisible = true
        }
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    okExecutionTime() {
      var id = this.selectedRowKeys.map(item => item)
      var pm = new URLSearchParams()
      pm.append('id', id)
      pm.append('startTime', this.executionTime.startTime)
      pm.append('endTime', this.executionTime.endTime)
      putAction(this.url.adjustExecutionTime, pm).then(res => {
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
        this.close()
        this.loadData()
      })
    },
    okJumpTime() {
      var id = this.selectedRowKeys.map(item => item)
      var pm = new URLSearchParams()
      pm.append('id', id)
      pm.append('skipTimeOne', this.jumpTimeData.skipTimeOne) //时间段1
      pm.append('skipTimeOneEnd', this.jumpTimeData.skipTimeOneEnd)
      pm.append('skipTimeTwo', this.jumpTimeData.skipTimeTwo)
      pm.append('skipTimeTwoEnd', this.jumpTimeData.skipTimeTwoEnd) //时间段2
      pm.append('skipWeekend', this.jumpTimeData.skipWeekend)
      pm.append('skipDate', this.jumpTimeData.skipDate) //跳过日期
      // let pm = this.jumpTimeData

      putAction(this.url.setSkipTime, pm).then(res => {
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
        this.close()
        this.loadData()
      })
    },
    handleAdds() {
      console.log(11)
      this.$refs.verifyModalForm.editor()
    },
    handleAdd() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '添加'
      this.$refs.modalForm.disableSubmit = false
      this.$refs.modalForm.disSubmit = false
      // 隐藏OK按钮
      // this.$refs.modalForm.disSubmit = true
    },
    handleReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleOk() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
      this.visible = false
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.operatingRecordList.details(record)
            this.$refs.archivedDocumentList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    // 执行保养
    handleExecute(record) {
      this.$refs.modalForm.execute(record)
      this.$refs.modalForm.title = '执行保养'
      this.$refs.modalForm.disableSubmit = false
    },
    handleScore(record) {
      console.log('record', record)
      // this.scoreParam = Object.assign({}, record)
      getAction('/keepCheck/tabCheckKeepOrder/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.scoreParam = res.result
          this.$forceUpdate()
        } else {
          this.$message.warning(res.message)
        }
      })
      this.scoreVisible = true
    },
    handleVerify() {
      if (this.selectedRowKeys.length == 0) {
        this.$message.warning('请选中一条数据！')
      } else {
        let dataList = this.selectionRows.map(item => item.keepProcessStatus)
        // let dataId = this.selectionRows.map(item => item.id)
        let verif = '待验证'
        let verifT = dataList.every(item => item === verif)
        if (verifT == true) {
          // console.log('数据全等')
          this.$refs.verifyModalForm.editor(this.selectionRows)
        } else {
          this.$message.warning('保养单状态不符！')
        }
        // httpAction(httpUrl, '', method).then(res => {
        //   if (res.success) {
        //     //重新计算分页问题
        //     this.reCalculatePage(this.selectedRowKeys.length)
        //     this.$message.success(res.message)
        //     this.loadData()
        //     this.onClearSelected()
        //   } else {
        //     this.$message.warning(res.message)
        //   }
        // })
        // this.$message.warning('保养单状态不符！')
      }
    },
    initDictConfig() {}
  }
}
</script>
<style lang="less" scoped>
.ant-radio-button-wrapper {
  margin: 0 10px;
}
</style>
