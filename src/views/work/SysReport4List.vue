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
      <!--  <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button v-has="'SysReport4List:export'" type="primary" icon="download" @click="handleExportXls('在制品报表')"
        >导出</a-button
      >
      <!-- <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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

        <!-- <span slot="action" slot-scope="text, record">
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
        </span> -->
      </a-table>
    </div>

    <sys-report4-modal ref="modalForm" @ok="modalFormOk"></sys-report4-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysReport4Modal from './modules/SysReport4Modal'

export default {
  name: 'SysReport4List',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysReport4Modal
  },
  data() {
    return {
      description: '生产报表管理页面',
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
          title: '工单编码',
          align: 'center',
          dataIndex: 'projectCode'
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
          dataIndex: 'productUnit'
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productSpec'
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
          dataIndex: 'workStatus'
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
          title: '工单备注',
          align: 'center',
          dataIndex: 'workRemark'
        },
        {
          title: '工序顺序',
          align: 'center'
          // dataIndex: 'processOrder'
        },
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'procedureName'
        },
        {
          title: '工序编号',
          align: 'center',
          dataIndex: 'procedureCode'
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
          title: '任务实际开始时间',
          align: 'center',
          dataIndex: 'taskRealStartTime',
          customRender: function(text) {
            return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          }
        },
        {
          title: '任务实际结束时间',
          align: 'center',
          dataIndex: 'taskRealEndTime',
          customRender: function(text) {
            return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          }
        },
        {
          title: '任务计划数',
          align: 'center',
          dataIndex: 'taskPlanAmount'
        },
        {
          title: '任务实际数',
          align: 'center',
          dataIndex: 'taskRealAmount'
        },
        {
          title: '生产人员',
          align: 'center'
          // dataIndex: 'operator'
        },
        {
          title: '标准效率',
          align: 'center'
          // dataIndex: 'standardEfficiency'
        },
        {
          title: '实际效率',
          align: 'center'
          // dataIndex: 'actualEfficiency'
        },
        {
          title: '达标率',
          align: 'center'
          // dataIndex: 'controlRate'
        },
        {
          title: '报工数',
          align: 'center',
          dataIndex: 'outputAmount'
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
          title: '不良品原因',
          align: 'center'
          // dataIndex: 'defectsstr'
        },
        {
          title: '报工开始时间',
          align: 'center'
          // dataIndex: 'reporStartTime',
          // customRender: function(text) {
          //   return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          // }
        },
        {
          title: '报工结束时间',
          align: 'center'
          // dataIndex: 'reporEndTime',
          // customRender: function(text) {
          //   return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          // }
        },
        {
          title: '实际报工时间',
          align: 'center'
          // dataIndex: 'reporRealTime',
          // customRender: function(text) {
          //   return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          // }
        },
        {
          title: '报工时长',
          align: 'center'
          // dataIndex: 'outputDuration'
        },
        {
          title: '计价方式',
          align: 'center'
          // dataIndex: 'pricingType'
        },
        {
          title: '工资单价',
          align: 'center'
          // dataIndex: 'unitPrice'
        },
        {
          title: '预计工资	',
          align: 'center'
          // dataIndex: 'expectedSalary'
        },
        {
          title: '审批人',
          align: 'center'
          // dataIndex: 'checkor'
        },
        {
          title: '审批时间',
          align: 'center'
          // dataIndex: 'checkTime',
          // customRender: function(text) {
          //   return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
          // }
        },
        {
          title: '审批状态',
          align: 'center'
          // dataIndex: 'checkStatus'
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
        }
      ],
      url: {
        list: '/longxianda/report/queryWip',
        delete: '/longxianda2/sysReport4/delete',
        deleteBatch: '/longxianda2/sysReport4/deleteBatch',
        exportXlsUrl: '/longxianda/report/wipExportXls',
        importExcelUrl: 'longxianda2/sysReport4/importExcel'
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
      fieldList.push({ type: 'string', value: 'productUnit', text: '产品单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productSpec', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workPlanAmount', text: '工单计划数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workRealaMount', text: '工单实际数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workStatus', text: '工单状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workStartTime', text: '工单计划开始时间', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workEndTime', text: '工单计划结束时间', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workRealStartTime', text: '工单实际开始时间', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workRealEndTime', text: '工单实际结束时间', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workRemark', text: '工单备注', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processOrder', text: '工序顺序', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processName', text: '工序名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processCode', text: '工序编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outputRate', text: '报工数配比', dictCode: '' })
      fieldList.push({ type: 'string', value: 'taskStatus', text: '任务状态', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'taskStartTime', text: '任务计划开始时间', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'taskEndTime', text: '任务计划结束时间', dictCode: '' })
      // fieldList.push({ type: 'date', value: 'taskRealStartTime', text: '任务实际开始时间' })
      // fieldList.push({ type: 'date', value: 'taskRealEndTime', text: '任务实际结束时间' })
      // fieldList.push({ type: 'int', value: 'taskPlanAmount', text: '任务计划数', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'taskRealAmount', text: '任务实际数', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'operator', text: '生产人员', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'standardEfficiency', text: '标准效率', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'actualEfficiency', text: '实际效率', dictCode: '' })
      // fieldList.push({ type: 'BigDecimal', value: 'controlRate', text: '达标率', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'outputAmount', text: '报工数', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'fineAmount', text: '良品数', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'defectsAmount', text: '不良品数', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'defectsstr', text: '不良品原因', dictCode: '' })
      // fieldList.push({ type: 'date', value: 'reporStartTime', text: '报工开始时间' })
      // fieldList.push({ type: 'date', value: 'reporEndTime', text: '报工结束时间' })
      // fieldList.push({ type: 'date', value: 'reporRealTime', text: '实际报工时间' })
      // fieldList.push({ type: 'int', value: 'outputDuration', text: '报工时长', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'pricingType', text: '计价方式', dictCode: '' })
      // fieldList.push({ type: 'BigDecimal', value: 'unitPrice', text: '工资单价', dictCode: '' })
      // fieldList.push({ type: 'BigDecimal', value: 'expectedSalary', text: '预计工资	', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'checkor', text: '审批人', dictCode: '' })
      // fieldList.push({ type: 'date', value: 'checkTime', text: '审批时间' })
      // fieldList.push({ type: 'string', value: 'checkStatus', text: '审批状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
