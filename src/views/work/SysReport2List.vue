<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <!-- <a-col :md="6" :sm="12">
            <a-form-item label="工单编号">
              <a-input placeholder="请输入工单编号" v-model="queryParam.projectCode" />
            </a-form-item>
          </a-col> -->
          <a-col :md="6" :sm="8">
            <a-form-item label="产品名称">
              <a-input placeholder="请输入产品名称" v-model="queryParam.productName" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="产品编号">
              <a-input placeholder="请输入" v-model="queryParam.productCode" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
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
          <a-col :md="6" :sm="8">
            <a-form-item label="工序">
              <a-select
                mode="multiple"
                placeholder="请选择"
                v-model="processCodes"
                @change="handleChangeProcedure"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option v-for="item in procedureOptions" :key="item.id" :value="item.code" :label="item.name">
                  {{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="报工开始时间">
              <a-range-picker v-model="queryTime1" @change="onChangeOutputStartTime" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="报工结束时间">
              <a-range-picker v-model="queryTime2" @change="onChangeOutputEndTime" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="实际报工时间">
              <a-range-picker v-model="queryTime3" @change="onChangeOutputTime" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="时间粒度">
              <a-select
                placeholder="请选择"
                v-model="queryTime"
                @change="handleChange"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option value="DAY" label="每天">
                  每天
                </a-select-option>
                <a-select-option value="WEEK" label="每周">
                  每周
                </a-select-option>
                <a-select-option value="MONTH" label="每月">
                  每月
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="业务维度">
              <a-select placeholder="请选择" v-model="queryParam.bizType" @change="handleChange">
                <a-select-option :value="1">
                  按成品
                </a-select-option>
                <a-select-option :value="2">
                  按工序
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" v-has="'SysReport2List:export'" @click="handleExportXls('产量统计报表')"
        >导出</a-button
      >
      <!-- <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->
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

    <sys-report2-modal ref="modalForm" @ok="modalFormOk"></sys-report2-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysReport2Modal from './modules/SysReport2Modal'
import { getAction } from '@api/manage'

export default {
  name: 'SysReport2List',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysReport2Modal
  },
  data() {
    return {
      description: '产品统计报表管理页面',
      producerIdsList: [],
      processCodes: [],
      queryTime: [],
      queryTime1: [],
      queryTime2: [],
      queryTime3: [],
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
        //   title: '工单编号',
        //   align: 'center',
        //   dataIndex: 'projectCode'
        // },
        // {
        //   title: '工单计划开始时间',
        //   align: 'center',
        //   dataIndex: 'planStartTime',
        //   customRender: function(text) {
        //     return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
        //   }
        // },
        // {
        //   title: '工单计划结束时间',
        //   align: 'center',
        //   dataIndex: 'planEndTime',
        //   customRender: function(text) {
        //     return !text ? '' : text.length > 10 ? text.substr(0, 10) : text
        //   }
        // },
        {
          title: '工单计划数',
          align: 'center',
          dataIndex: 'planAmount'
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
          title: '产品属性',
          align: 'center',
          dataIndex: 'productProperty'
        },
        {
          title: '工艺路线',
          align: 'center',
          dataIndex: 'craftRouteId'
        },
        {
          title: '单位',
          align: 'center',
          dataIndex: 'productUnit'
        },
        // {
        //   title:'工序',
        //   align:"center",
        //   dataIndex: 'processCode'
        // },
        // {
        //   title:'工序名称',
        //   align:"center",
        //   dataIndex: 'processName'
        // },
        {
          title: '合计',
          align: 'center',
          dataIndex: 'amount'
        }
      ],
      url: {
        procedureList: '/longxianda/sysProcedure/list',
        list: 'longxianda/report/queryOutputCount',
        delete: '/longxianda2/sysReport2/delete',
        deleteBatch: '/longxianda2/sysReport2/deleteBatch',
        exportXlsUrl: '/longxianda/report/outputCountExportXls',
        importExcelUrl: 'longxianda2/sysReport2/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      procedureOptions: []
    }
  },
  created() {
    this.init()
    this.getSuperFieldList()
    this.getProcedureList()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    // searchQuery() {
    //   var producerIds = ''
    //   for (var a = 0; a < this.queryParam.producerIds.length; a++) {
    //     producerIds += this.queryParam.producerIds[a] + ','
    //   }
    //   this.queryParam.producerIds = producerIds
    //   this.loadData(1)
    //   // 点击查询清空列表选中行
    //   this.selectedRowKeys = []
    //   this.selectionRows = []
    // },
    searchReset() {
      this.queryTime = []
      this.queryTime1 = []
      this.queryTime2 = []
      this.queryTime3 = []
      this.queryParam = {}
      this.processCodes = []
      // this.queryParam.processCodes = {}
      this.loadData(1)
    },
    init() {
      getAction('/sys/dict/getDictItems/sys_user,realname,id').then(res => {
        this.producerIdsList = res.result
      })
    },
    initDictConfig() {},
    getProcedureList() {
      if (!this.url.procedureList) {
        this.$message.error('请设置url.procedureList属性!')
        return
      }
      const params = {}
      params.pageNo = 1
      params.pageSize = 1000
      getAction(this.url.procedureList, params)
        .then(res => {
          if (res.success) {
            this.procedureOptions = res.result.records
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {})
    },
    getSuperFieldList() {
      let fieldList = []
      // fieldList.push({ type: 'string', value: 'projectCode', text: '工单编号', dictCode: '' })
      fieldList.push({ type: 'date', value: 'planStartTime', text: '工单计划开始时间' })
      fieldList.push({ type: 'date', value: 'planEndTime', text: '工单计划结束时间' })
      fieldList.push({ type: 'int', value: 'planAmount', text: '工单计划数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productSpec', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productUnit', text: '单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processCode', text: '工序', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processName', text: '工序名称', dictCode: '' })
      fieldList.push({ type: 'int', value: 'amount', text: '合计', dictCode: '' })
      this.superFieldList = fieldList
    },
    onChangeOutputStartTime(date, dateString) {
      this.queryTime1 = date
      this.queryParam.outputStartTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputStartTimeTo = dateString[0] ? dateString[1] + ' 00:00:00' : ''
    },
    onChangeOutputEndTime(date, dateString) {
      this.queryTime2 = date
      this.queryParam.outputEndTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputEndTimeTo = dateString[1] ? dateString[1] + ' 00:00:00' : ''
    },
    onChangeOutputTime(date, dateString) {
      this.queryTime3 = date
      this.queryParam.outputTimeFrom = dateString[0] ? dateString[0] + ' 00:00:00' : ''
      this.queryParam.outputTimeTo = dateString[1] ? dateString[1] + ' 00:00:00' : ''
    },
    handleChange(value) {
      if (typeof value === 'number') {
        this.queryParam.bizType = value
      } else {
        this.queryParam.granularity = value
      }
    },
    handleChangeProcedure(value) {
      var ids = ''
      for (var a = 0; a < value.length; a++) {
        ids += value[a] + ','
      }
      this.queryParam.processCodes = value
      // this.queryParam.processCodes = ids
      // this.queryParam.processCodes = value.length ? encodeURIComponent('[' + value.toString() + ']') : ''
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
