<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="设备编号">
              <a-input v-model.trim="queryParam.deviceCode" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备名称">
              <a-input v-model.trim="queryParam.deviceName" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="点检时间">
                <!-- <a-input v-model.trim="queryParam.sysOrgCode" placeholder="请输入"> </a-input> -->
                <a-range-picker v-model="queryTime" @change="onChangeRangeDate" format="YYYY-MM-DD" />
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="点检人">
                <!-- <a-input v-model.trim="queryParam.spotCheckItemId" placeholder="请输入"> </a-input> -->
                <j-dict-select-tag
                  v-model="queryParam.spotCheckItemId"
                  placeholder="请选择点检人"
                  dictCode="sys_user,realname,id"
                />
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
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button @click="handleRecord" type="primary" icon="plus">添加记录</a-button>
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
      <a-tab-pane tab="点检明细" key="0">
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
      <a-tab-pane tab="点检记录" key="1">
        <Check-record-List ref="checkRecordList"></Check-record-List>
      </a-tab-pane>
    </a-tabs>
    <!--新增、编辑 -->
    <tab-spot-check-items-modal ref="modalForm" @ok="modalFormOk"></tab-spot-check-items-modal>
    <!--添加记录 -->
    <Record-modal ref="recordModal" @ok="modalFormOk"></Record-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import RecordModal from './modules/recordModal'
import TabSpotCheckItemsModal from './modules/TabSpotCheckItemsModal'
import OperatingRecordList from './modules/operatingRecordList'
import CheckRecordList from './modules/checkRecordList'

export default {
  name: 'TabSpotCheckItemsList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    OperatingRecordList,
    CheckRecordList,
    RecordModal,
    TabSpotCheckItemsModal
  },
  data() {
    return {
      description: '点检项表管理页面',
      // 表头
      queryTime: [],
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
          title: '开机点检项',
          align: 'center',
          dataIndex: 'patrolItem'
        },
        {
          title: '上次点检时间',
          align: 'center',
          dataIndex: 'lastSpotCheckTime'
        },
        {
          title: '操作人',
          align: 'center',
          dataIndex: 'operatorId'
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
        list: '/spot/tabSpotCheckItems/list',
        delete: '/spot/tabSpotCheckItems/delete',
        deleteBatch: '/spot/tabSpotCheckItems/deleteBatch',
        exportXlsUrl: '/spot/tabSpotCheckItems/exportXls',
        importExcelUrl: 'spot/tabSpotCheckItems/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {},
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    //
    handleEdit: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '编辑'
      this.$refs.modalForm.disableSubmit = false
    },
    // 添加记录
    handleRecord() {
      let record = this.selectionRows
      // console.log('record', record)
      if (record.length == 1) {
        this.$refs.recordModal.edit(record)
        this.$refs.recordModal.title = '添加记录'
        this.$refs.recordModal.disableSubmit = false
      } else {
        this.$message.warning('请选择一条数据!')
      }
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            // this.$refs.operatingRecordList.details(record)
            // console.log('checkRecordList', record)
            this.$refs.checkRecordList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.lastSpotCheckStartTime = dateString[0]
      this.queryParam.lastSpotCheckEndTime = dateString[1]
      console.log('this.queryParam.lastSpotCheckEndTime', this.queryParam)
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
