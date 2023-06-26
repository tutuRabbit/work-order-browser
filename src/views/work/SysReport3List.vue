<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button v-has="'SysReport3List:export'" type="primary" icon="download" @click="handleExportXls('生产报表')"
        >导出</a-button
      >
      <!--  <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <!-- <a-dropdown v-if="selectedRowKeys.length > 0">
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
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text, record">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img
            v-else
            :src="getImgView(text)"
            :preview="record.id"
            height="25px"
            alt=""
            style="max-width:80px;font-size: 12px;font-style: italic;"
          />
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button v-else :ghost="true" type="primary" icon="download" size="small" @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <!--  <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
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
 -->
      </a-table>
    </div>

    <sys-report3-modal ref="modalForm" @ok="modalFormOk"></sys-report3-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysReport3Modal from './modules/SysReport3Modal'

export default {
  name: 'SysReport3List',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysReport3Modal
  },
  data() {
    return {
      description: '在制品报表管理页面',
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
          title: '产品编码',
          align: 'center',
          dataIndex: 'productCode'
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '产品单位',
          align: 'center',
          dataIndex: 'productSpec'
        },
        {
          title: '产品规则',
          align: 'center',
          dataIndex: 'productUnit'
        },
        {
          title: '存放厂房',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '成本单价',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '类别',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '重量',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '工单编码',
          align: 'center',
          dataIndex: 'projectCode'
        },
        {
          title: '工单计划数',
          align: 'center',
          dataIndex: 'workPlanAmount'
        },
        {
          title: '工单实际数',
          align: 'center',
          dataIndex: 'workRealAmount'
        },
        {
          title: '工单状态',
          align: 'center',
          dataIndex: 'workStatus',
          customRender: function(text) {
            if (text === '0') {
              return '未开始'
            } else if (text === '10') {
              return '执行中'
            } else if (text === '100') {
              return '已结束'
            } else {
              return '-'
            }
          }
        },
        {
          title: '工单计划开始时间',
          align: 'center',
          dataIndex: 'workStartTime'
        },
        {
          title: '工单计划结束时间',
          align: 'center',
          dataIndex: 'workEndTime'
        },
        {
          title: '工单实际开始时间',
          align: 'center',
          dataIndex: 'workRealStartTime'
        },
        {
          title: '工单实际结束时间',
          align: 'center',
          dataIndex: 'workRealEndTime'
        },
        {
          title: '工单类型',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '工单备注',
          align: 'center',
          dataIndex: 'workRemark'
        },
        {
          title: '产品信息',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '客户名称',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '客户类型',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '是否启动微信通知',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '车间类型',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '重量',
          align: 'center'
          // dataIndex: 'productUnit'
        },
        {
          title: '工序顺序',
          align: 'center',
          dataIndex: 'processOrder'
        },
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'processName'
        },
        {
          title: '工序编号',
          align: 'center',
          dataIndex: 'processCode'
        },
        {
          title: '报工数配比',
          align: 'center',
          dataIndex: 'outputRate'
        },
        {
          title: '任务状态',
          align: 'center',
          dataIndex: 'taskStatus'
        },
        {
          title: '任务计划开始时间',
          align: 'center',
          dataIndex: 'taskStartTime'
        },
        {
          title: '任务计划结束时间',
          align: 'center',
          dataIndex: 'taskEndTime'
        },
        {
          title: '任务计划数',
          align: 'center',
          dataIndex: 'taskEndTime'
        },
        {
          title: '任务实际数',
          align: 'center',
          dataIndex: 'taskRealAmount'
        },
        {
          title: '技术要求',
          align: 'center',
          dataIndex: 'taskEndTime'
        },
        {
          title: '生产人员',
          align: 'center',
          dataIndex: 'operator'
        },
        {
          title: '标准效率',
          align: 'center',
          dataIndex: 'standardEfficiency'
        },
        {
          title: '实际效率',
          align: 'center',
          dataIndex: 'actualEfficiency'
        },
        {
          title: '达标率',
          align: 'center',
          dataIndex: 'controlRate'
        },
        {
          title: '报工数',
          align: 'center',
          dataIndex: 'outputNumber'
        },
        {
          title: '良品数',
          align: 'center',
          dataIndex: 'fineNumber'
        },
        {
          title: '不良品数',
          align: 'center',
          dataIndex: 'defectsNumber'
        },
        {
          title: '审批时间',
          align: 'center',
          dataIndex: 'checkTime'
        },
        {
          title: '不良品原因',
          align: 'center',
          dataIndex: 'defectsstr'
        },
        {
          title: '报工开始时间',
          align: 'center',
          dataIndex: 'reporStartTime'
        },
        {
          title: '报工结束时间',
          align: 'center',
          dataIndex: 'reporEndTime'
        },
        {
          title: '报工时长',
          align: 'center',
          dataIndex: 'outputDuration'
        },
        {
          title: '计价方式',
          align: 'center',
          dataIndex: 'pricingType'
        },
        {
          title: '工资单价',
          align: 'center',
          dataIndex: 'unitPrice'
        },
        {
          title: '预计工资',
          align: 'center',
          dataIndex: 'expectedSalary'
        },
        {
          title: '审批状态',
          align: 'center',
          dataIndex: 'checkStatus',
          customRender: function(text) {
            if (text === '0') {
              return '未审批'
            } else if (text === '1') {
              return '已审批'
            } else {
              return '-'
            }
          }
        },
        {
          title: '审批人',
          align: 'center',
          dataIndex: 'checkor'
        },
        {
          title: '实际报工时间',
          align: 'center',
          dataIndex: 'reporRealTime'
        },
        {
          title: '报工创建人',
          align: 'center',
          dataIndex: 'reportCreateName'
        },
        {
          title: '报工修改人',
          align: 'center',
          dataIndex: 'reportUpdateName'
        },
        {
          title: '不良备注原因',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '报工类型',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '晚班工时',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '每件钻孔个数',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '白班工时',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '部门名称',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '五险一金扣除金额',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '入职时间',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '发放工资次数',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '基本工资',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '职位类型',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '请假1小时扣除金额',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '请假小时',
          align: 'center'
          // dataIndex: 'defectsstr'
        }
      ],
      url: {
        list: '/longxianda/report/queryProduce',
        delete: '/longxianda2/sysReport3/delete',
        deleteBatch: '/longxianda2/sysReport3/deleteBatch',
        exportXlsUrl: '/longxianda/report/produceExportXls',
        importExcelUrl: 'longxianda/report/produceExportXls'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'projectCode', text: '工单编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productSpec', text: '产品单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productUnit', text: '产品规则', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
