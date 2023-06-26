<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="4" :sm="12">
            <a-form-item label="年度保养计划">
              <j-dict-select-tag
                type="list"
                dictCode="year_plan_yearth"
                v-model.trim="queryParam.keepYearth"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="使用部门">
              <a-tree-select
                v-model.trim="queryParam.sysOrgCode"
                style="width: 100%"
                :tree-data="treeData"
                placeholder="请选择"
              >
              </a-tree-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="12">
            <a-form-item label="设备类别">
              <j-category-select v-model="queryParam.deviceType" pcode="B03" />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="12">
              <a-form-item label="快速查询">
                <a-input v-model="queryParam.deviceCodeAndName" placeholder="设备名称，设备编码" />
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="visible = true" icon="filter" style="margin-left: 8px">更多</a-button>
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
      <a-button @click="handleAdd" type="primary" icon="plus">定制年度计划</a-button>
      <a-button type="primary" icon="reload" @click="handleSwitchMaintain">转保养计划</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('保养年计划')">导出</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
      <span style="display:black;margin-left:10px">
        <img src="@/assets/year-plan/1.png" />
        已完成
      </span>
      <span style="display:black;margin-left:10px">
        <img src="@/assets/year-plan/3.png" />
        实施中
      </span>
      <span style="display:black;margin-left:10px">
        <img src="@/assets/year-plan/2.png" />
        待实施
      </span>
      <span style="display:black;margin-left:10px">
        <img src="@/assets/year-plan/0.png" />
        未开始
      </span>
      <span style="display:black;margin-left:10px">
        <img src="@/assets/year-plan/4.png" />
        已关闭
      </span>
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
      >
        <template slot="iconSlot1" slot-scope="text, record">
          <img v-if="record.month1 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month1 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month1 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month1 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month1 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot2" slot-scope="text, record">
          <img v-if="record.month2 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month2 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month2 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month2 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month2 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot3" slot-scope="text, record">
          <img v-if="record.month3 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month3 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month3 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month3 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month3 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot4" slot-scope="text, record">
          <img v-if="record.month4 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month4 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month4 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month4 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month4 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot5" slot-scope="text, record">
          <img v-if="record.month5 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month5 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month5 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month5 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month5 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot6" slot-scope="text, record">
          <img v-if="record.month6 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month6 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month6 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month6 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month6 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot7" slot-scope="text, record">
          <img v-if="record.month7 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month7 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month7 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month7 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month7 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot8" slot-scope="text, record">
          <img v-if="record.month8 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month8 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month8 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month8 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month8 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot9" slot-scope="text, record">
          <img v-if="record.month9 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month9 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month9 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month9 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month9 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot10" slot-scope="text, record">
          <img v-if="record.month10 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month10 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month10 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month10 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month10 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot11" slot-scope="text, record">
          <img v-if="record.month11 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month11 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month11 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month11 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month11 == '4'" src="@/assets/year-plan/1.png" />
        </template>
        <template slot="iconSlot12" slot-scope="text, record">
          <img v-if="record.month12 == '0'" src="@/assets/year-plan/4.png" />
          <img v-if="record.month12 == '1'" src="@/assets/year-plan/0.png" />
          <img v-if="record.month12 == '2'" src="@/assets/year-plan/2.png" />
          <img v-if="record.month12 == '3'" src="@/assets/year-plan/3.png" />
          <img v-if="record.month12 == '4'" src="@/assets/year-plan/1.png" />
        </template>
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

    <tab-keep-year-plan-modal ref="modalForm" @ok="modalFormOk"></tab-keep-year-plan-modal>
    <!-- 转保养计划 -->
    <maintain-modal-form ref="maintainModalForm" @ok="modalFormOk"></maintain-modal-form>
    <!-- 高级查询 -->
    <j-modal title="更多查询" width="50%" :visible="visible" @cancel="visible = false" cancelText="关闭">
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
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
              <a-form-item label="保养月份">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="queryParam.keepMonth"
                  dictCode="keep_year_plan_months"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备标记">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="queryParam.deviceMark"
                  dictCode="keep_year_plan_months"
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
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabKeepYearPlanModal from './modules/TabKeepYearPlanModal'
import { putAction, getFileAccessHttpUrl } from '@/api/manage'
import maintainModalForm from './modules/maintainModalForm'
import { queryDepartTreeList } from '@/api/api'

export default {
  name: 'TabKeepYearPlanList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    maintainModalForm,
    TabKeepYearPlanModal
  },
  data() {
    return {
      description: '保养年计划管理页面',
      treeData: [],
      visible: false,
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
          title: '设备编号',
          align: 'center',
          dataIndex: 'deviceCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'deviceName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'deviceModel'
        },
        {
          title: '设备类型',
          align: 'center',
          dataIndex: 'deviceType'
        },
        {
          title: '使用部门',
          align: 'center',
          dataIndex: 'sysOrgCode'
        },
        {
          title: '保养级别',
          align: 'center',
          dataIndex: 'keepLevel'
        },
        {
          title: '保养标准',
          align: 'center',
          dataIndex: 'keepStandard'
        },
        {
          title: '保养时间',
          align: 'center',
          dataIndex: 'keepTime'
        },
        {
          title: '保养年份',
          align: 'center',
          dataIndex: 'keepYearth'
        },
        {
          title: '1月',
          align: 'center',
          dataIndex: 'month1',
          scopedSlots: { customRender: 'iconSlot1' }
        },
        {
          title: '2月',
          align: 'center',
          dataIndex: 'month2',
          scopedSlots: { customRender: 'iconSlot2' }
        },
        {
          title: '3月',
          align: 'center',
          dataIndex: 'month3',
          scopedSlots: { customRender: 'iconSlot3' }
        },
        {
          title: '4月',
          align: 'center',
          dataIndex: 'month4',
          scopedSlots: { customRender: 'iconSlot4' }
        },
        {
          title: '5月',
          align: 'center',
          dataIndex: 'month5',
          scopedSlots: { customRender: 'iconSlot5' }
        },
        {
          title: '6月',
          align: 'center',
          dataIndex: 'month6',
          scopedSlots: { customRender: 'iconSlot6' }
        },
        {
          title: '7月',
          align: 'center',
          dataIndex: 'month7',
          scopedSlots: { customRender: 'iconSlot7' }
        },
        {
          title: '8月',
          align: 'center',
          dataIndex: 'month8',
          scopedSlots: { customRender: 'iconSlot8' }
        },
        {
          title: '9月',
          align: 'center',
          dataIndex: 'month9',
          scopedSlots: { customRender: 'iconSlot9' }
        },
        {
          title: '10月',
          align: 'center',
          dataIndex: 'month10',
          scopedSlots: { customRender: 'iconSlot10' }
        },
        {
          title: '11月',
          align: 'center',
          dataIndex: 'month11',
          scopedSlots: { customRender: 'iconSlot11' }
        },
        {
          title: '12月',
          align: 'center',
          dataIndex: 'month12',
          scopedSlots: { customRender: 'iconSlot12' }
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
        list: '/year/tabKeepYearPlan/list',
        delete: '/year/tabKeepYearPlan/delete',
        deleteBatch: '/year/tabKeepYearPlan/deleteBatch',
        exportXlsUrl: '/year/tabKeepYearPlan/exportXls',
        importExcelUrl: 'year/tabKeepYearPlan/importExcel'
      },
      notHave: '@/assets/year-plan/1.png',
      dictOptions: {},
      superFieldList: [],
      rowList: []
    }
  },
  created() {
    let date = new Date()
    this.queryParam.keepYearth = date.getFullYear() //获取完整的年份(4位)
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  computed: {},
  methods: {
    handleDetail: function(record) {
      this.$refs.modalForm.show(record)
      this.$refs.modalForm.title = '详情'
      this.$refs.modalForm.disableSubmit = true
    },
    handleReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleOk() {
      this.loadData(1)
      // 点击查询清空列表选中行
      // https://gitee.com/jeecg/jeecg-boot/issues/I4KTU1
      this.selectedRowKeys = []
      this.selectionRows = []
      this.visible = false
    },
    handleAdd() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '定制年度计划'
      this.$refs.modalForm.disableSubmit = false
    },
    handleSwitchMaintain() {
      if (this.selectedRowKeys.length > 0) {
        this.rowList = []
        this.selectionRows.forEach((item, i) => {
          if (item.month1 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '1月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month2 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '2月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month3 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '3月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month4 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '4月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month5 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '5月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month6 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '6月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month7 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '7月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month8 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '8月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month9 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '9月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month10 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '10月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month11 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '11月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
          if (item.month12 == '2') {
            let pm = {
              yearPlanId: item.id,
              startTime: item.startTime,
              endTime: item.endTime,
              keepManager: item.keepManager,
              planMonth: item.planMonth,
              deviceId: item.deviceIds,
              deviceCode: item.deviceCode,
              deviceName: item.deviceName,
              deviceModel: item.deviceModel,
              deviceType: item.deviceType,
              sysOrgCode: item.sysOrgCode,
              keepLevel: item.keepLevel,
              keepYearth: item.keepYearth,
              keepStandard: item.keepStandard,
              planMonth: '12月',
              id: Math.round(Math.random() * 10)
            }
            this.rowList.push(pm)
          }
        })
        // console.log('this.rowList', this.rowList)
        // console.log('this.selectionRows', this.selectionRows)
        this.$refs.maintainModalForm.editor(this.rowList)
      } else {
        this.$message.warning('请至少选择一条计划执行转保养计划操作！')
      }
    },
    initDictConfig() {}
    // getSuperFieldList() {
    //   let fieldList = []
    //   fieldList.push({ type: 'string', value: 'deviceId', text: '设备id', dictCode: '' })
    //   fieldList.push({ type: 'string', value: 'keepLevel', text: '保养级别id', dictCode: '' })
    //   fieldList.push({ type: 'string', value: 'keepStandard', text: '保养标准', dictCode: '' })
    //   fieldList.push({ type: 'string', value: 'keepTime', text: '保养时间', dictCode: '' })
    // fieldList.push({ type: 'string', value: 'keepMonths', text: '保养月份', dictCode: '' })
    //   fieldList.push({ type: 'string', value: 'keepYearth', text: '保养计划年份', dictCode: '' })
    //   this.superFieldList = fieldList
    // }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
