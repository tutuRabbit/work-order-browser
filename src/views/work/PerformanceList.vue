<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <!-- <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="生产人名称">
              <a-input placeholder="请输入生产人名称" v-model="queryParam.userName"></a-input>
            </a-form-item>
          </a-col> -->
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="报工开始时间">
              <a-range-picker
                v-model="outputStartTime"
                style="width:100%"
                format="YYYY-MM-DD"
                :placeholder="['开始时间', '结束时间']"
                @change="outputStartChange"
              />
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="报工结束时间">
              <a-range-picker
                v-model="outputEndTime"
                style="width:100%"
                allowClear
                format="YYYY-MM-DD"
                :placeholder="['开始时间', '结束时间']"
                @change="outputEndChange"
              />
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="实际报工时间">
              <a-range-picker
                v-model="outputTime"
                allowClear
                style="width:100%"
                format="YYYY-MM-DD"
                :placeholder="['开始时间', '结束时间']"
                @change="outputTimeChange"
              />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :lg="7" :md="8" :sm="24">
              <a-form-item label="生产人员">
                <a-select
                  style="width: 100%"
                  mode="multiple"
                  placeholder="请选择生产人员"
                  v-model="queryParam.producerIds"
                  :allowClear="true"
                  :showSearch="true"
                  optionFilterProp="label"
                >
                  <a-select-option
                    v-for="(item, index) in producerIdsList"
                    :key="index"
                    :value="item.value"
                    :label="item.title"
                    >{{ item.title }}</a-select-option
                  >
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :lg="7" :md="8" :sm="24">
              <a-form-item label="工序">
                <a-select
                  style="width: 100%"
                  mode="multiple"
                  placeholder="请选择工序"
                  v-model="queryParam.processCodes"
                  :allowClear="true"
                  :showSearch="true"
                  optionFilterProp="label"
                >
                  <a-select-option
                    v-for="(item, index) in processCodesList"
                    :key="index"
                    :value="item.code"
                    :label="item.name"
                    >{{ item.name }}</a-select-option
                  >
                </a-select>
              </a-form-item>
            </a-col>
            <!-- <a-col :lg="7" :md="8" :sm="24">
              <a-form-item label="工序">
                <a-input placeholder="请选择工序" v-model="queryParam.processCodes"></a-input>
              </a-form-item>
            </a-col> -->
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
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

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!--  <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button v-has="'PerformanceList:export'" type="primary" icon="download" @click="handleExportXls('员工绩效报表')"
        >导出</a-button
      >
      <!-- <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown> -->
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
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <!-- <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span> -->
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <performance-modal ref="modalForm" @ok="modalFormOk"></performance-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import PerformanceModal from './modules/PerformanceModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '../../api/manage'

export default {
  name: 'PerformanceList',
  mixins: [JeecgListMixin],
  components: {
    PerformanceModal
  },
  data() {
    return {
      description: '员工绩效报表管理页面',
      // 表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        // {
        //        title: '生产人ID',
        //        align:"center",
        //        dataIndex: 'userId'
        //       },
        {
          title: '生产人名称',
          align: 'center',
          dataIndex: 'userName'
        },
        {
          title: '工序编号',
          align: 'center',
          dataIndex: 'processCode'
        },
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'processName'
        },
        {
          title: '产品编号',
          align: 'center',
          dataIndex: 'productCode'
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productSpec'
        },
        {
          title: '产品单位',
          align: 'center',
          dataIndex: 'productUnit'
        },
        {
          title: '良品数',
          align: 'center',
          dataIndex: 'fineAmount'
        },
        {
          title: '不良品数',
          align: 'center',
          dataIndex: 'defectsAmount'
        },
        {
          title: '合格率',
          align: 'center',
          dataIndex: 'defectRatioStr'
        },
        {
          title: '工资单价',
          align: 'center',
          dataIndex: 'unitPrice'
        },
        {
          title: '工资小计',
          align: 'center',
          dataIndex: 'salaryCount'
        },
        {
          title: '合计',
          align: 'center',
          dataIndex: 'totalAmount'
        }
        // {
        //   title: '操作',
        //   dataIndex: 'action',
        //   align: 'center',
        //   scopedSlots: { customRender: 'action' }
        // }
      ],
      url: {
        list: '/longxianda/report/queryPerformance',
        delete: '/longxianda/performance/delete',
        deleteBatch: '/longxianda/performance/deleteBatch',
        exportXlsUrl: '/longxianda/report/performanceExportXls',
        importExcelUrl: 'longxianda/performance/importExcel'
      },
      // 分页查询
      producerIdsList: [],
      processCodesList: [],
      outputStartTime: [],
      outputEndTime: [],
      outputTime: []
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  created() {
    this.init()
  },
  methods: {
    init() {
      getAction('/sys/dict/getDictItems/sys_user,realname,id').then(res => {
        this.producerIdsList = res.result
      })
      getAction('/longxianda/sysProcedure/list', { pageNo: 1, pageSize: 9999 }).then(res => {
        this.processCodesList = res.result.records
        console.log('this.processCodesList', this.processCodesList)
      })
    },
    searchReset() {
      this.outputStartTime = []
      this.outputEndTime = []
      this.outputTime = []
      this.queryParam = {}
      this.loadData(1)
    },
    outputStartChange(value, dateString) {
      this.outputStartTime = value
      this.queryParam.outputStartTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputStartTimeTo = dateString[1] ? dateString[1] + ' 00:00:00' : ''
    },
    outputEndChange(value, dateString) {
      this.outputEndTime = value
      this.queryParam.outputEndTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputEndTimeTo = dateString[1] ? dateString[1] + ' 00:00:00' : ''
    },
    outputTimeChange(value, dateString) {
      this.outputTime = value
      this.queryParam.outputTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputTimeTo = dateString[1] ? dateString[1] + ' 00:00:00' : ''
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
