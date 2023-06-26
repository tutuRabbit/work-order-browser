<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="登录账号">
              <a-input placeholder="请输入登录账号" v-model="queryParam.username"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="真实姓名">
              <a-input placeholder="请输入真实姓名" v-model="queryParam.realname"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="未审批良品数">
                <a-input placeholder="请输入未审批良品数" v-model="queryParam.outputNoApproveCount"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="未审批计件工资">
                <a-input placeholder="请输入未审批计件工资" v-model="queryParam.salaryByNoteNoApprove"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="未审批计时工资">
                <a-input placeholder="请输入未审批计时工资" v-model="queryParam.salaryByTimingNoApprove"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!-- <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a> -->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('工资报表')" v-has="'SalaryList:export'"
        >导出</a-button
      >
      <!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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
        <!--        <span slot="action" slot-scope="text, record">
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
    <salary-modal ref="modalForm" @ok="modalFormOk"></salary-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import SalaryModal from './modules/SalaryModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

export default {
  name: 'SalaryList',
  mixins: [JeecgListMixin],
  components: {
    SalaryModal
  },
  data() {
    return {
      description: '工资报表管理页面',
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
        {
          title: '登录账号',
          align: 'center',
          // dataIndex: 'userId'
          dataIndex: 'username'
        },
        {
          title: '真实姓名',
          align: 'center',
          // dataIndex: 'userName'
          dataIndex: 'realname'
        },
        {
          title: '未审批良品数',
          align: 'center',
          dataIndex: 'outputNoApproveCount'
        },
        {
          title: '未审批计件工资',
          align: 'center',
          dataIndex: 'salaryByNoteNoApprove'
        },
        {
          title: '未审批计时工资',
          align: 'center',
          dataIndex: 'salaryByTimingNoApprove'
        },
        {
          title: '已审批良品数',
          align: 'center',
          dataIndex: 'outputApproveCount'
        },
        {
          title: '已审批计件工资',
          align: 'center',
          dataIndex: 'salaryByNoteApprove'
        },
        {
          title: '已审批计时工资',
          align: 'center',
          dataIndex: 'salaryByTimingApprove'
        },
        {
          title: '良品总数',
          align: 'center',
          dataIndex: 'outputCount'
        },
        {
          title: '工资小计',
          align: 'center',
          dataIndex: 'salaryCount'
        }
        // {
        //   title: '操作',
        //   dataIndex: 'action',
        //   align:"center",
        //   scopedSlots: { customRender: 'action' },
        // }
      ],
      url: {
        list: '/longxianda/report/querySalary',
        delete: '/longxianda/salary/delete',
        deleteBatch: '/longxianda/salary/deleteBatch',
        exportXlsUrl: '/longxianda/report/salaryExportXls',
        importExcelUrl: 'longxianda/salary/importExcel'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {}
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
