<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="送检时间">
              <a-range-picker
                v-model="queryTime"
                style="width:100%"
                :show-time="{ format: 'HH:mm' }"
                @change="onChangeRangeDate"
                format="YYYY-MM-DD HH:mm:ss"
              />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备编号">
              <a-input v-model.trim="queryParam.deviceCode" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="设备名称">
                <a-input v-model.trim="queryParam.deviceName" placeholder="请输入"> </a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="单号">
                <a-input v-model.trim="queryParam.checkoutPlanRecordCode" placeholder="请输入"> </a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="检验方式">
                <a-select
                  :showSearch="true"
                  optionFilterProp="label"
                  v-model.trim="queryParam.checkWay"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="" label="全部">全部</a-select-option>
                  <a-select-option value="1" label="内检">内检</a-select-option>
                  <a-select-option value="2" label="外检">外检</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="检验类型">
                <a-select
                  :showSearch="true"
                  optionFilterProp="label"
                  v-model.trim="queryParam.checkType"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="" label="全部">全部</a-select-option>
                  <a-select-option value="1" label="定期检验">定期检验</a-select-option>
                  <a-select-option value="2" label="监督检验">监督检验</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="检验状态">
                <a-select
                  :showSearch="true"
                  optionFilterProp="label"
                  v-model.trim="queryParam.checkStatus"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="" label="全部">全部</a-select-option>
                  <a-select-option value="1" label="在检">在检</a-select-option>
                  <a-select-option value="2" label="结束">结束</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
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
      <a-button @click="handleAdd" type="primary">添加记录</a-button>
      <a-button type="primary" icon="import" @click="handleExportXls('检验记录')">导出</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
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
      <a-tab-pane tab="检验信息" key="0">
        <Check-record-List ref="checkRecordList"></Check-record-List>
      </a-tab-pane>
    </a-tabs>
    <!--新增、编辑 -->
    <tab-spot-check-items-modal ref="modalForm" @ok="modalFormOk"></tab-spot-check-items-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabSpotCheckItemsModal from './modules/TabSpotCheckItemsModal'
import CheckRecordList from './modules/checkRecordList'

export default {
  name: 'TabSpotCheckItemsList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    CheckRecordList,
    TabSpotCheckItemsModal
  },
  data() {
    return {
      description: '巡检记录管理页面',
      queryTime: [],
      queryParam: {
        checkWay: '',
        checkType: '',
        checkStatus: ''
      },
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
          title: '单号',
          align: 'center',
          dataIndex: 'checkoutPlanRecordCode'
        },
        {
          title: '检验方式',
          align: 'center',
          dataIndex: 'checkWay'
        },
        {
          title: '检验类型',
          align: 'center',
          dataIndex: 'checkType'
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'checkStatus'
        },
        {
          title: '送检人',
          align: 'center',
          dataIndex: 'submissionPerson'
        },
        {
          title: '送检时间',
          align: 'center',
          dataIndex: 'submissionTime'
        },
        {
          title: '检验员',
          align: 'center',
          dataIndex: 'checkoutPerson'
        },
        {
          title: '检验单位',
          align: 'center',
          dataIndex: 'checkoutUnit'
        },
        {
          title: '电话',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '总费用',
          align: 'center',
          dataIndex: 'totalMoney'
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
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
        list: '/check_plan_record/tabCheckoutPlanRecords/list',
        delete: '/check_plan_record/tabCheckoutPlanRecords/delete',
        deleteBatch: '/check_plan_record/tabCheckoutPlanRecords/deleteBatch',
        // 导出检验计划记录
        exportXlsUrl: '/check_plan_record/tabCheckoutPlanRecords/importCheckPlanRecordsExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {},
  methods: {
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.startTime = dateString[0]
      this.queryParam.endTime = dateString[1]
    },
    handleEdit: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '编辑'
      this.$refs.modalForm.disableSubmit = false
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            console.log('record', record)
            this.$refs.checkRecordList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
