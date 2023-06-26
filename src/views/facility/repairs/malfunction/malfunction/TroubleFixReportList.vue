<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="处理状态">
              <a-select
                v-model.trim="queryParam.dealStatus"
                :showSearch="true"
                optionFilterProp="label"
                style="width: 100%"
              >
                <a-select-option value="" label="全部">全部</a-select-option>
                <a-select-option value="0" label="待维修">待维修</a-select-option>
                <a-select-option value="1" label="维修中">维修中</a-select-option>
                <a-select-option value="2" label="已完成">已完成</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="报修人">
              <j-dict-select-tag
                type="list"
                v-model.trim="queryParam.createBy"
                dictCode="sys_user,realname,id"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="提出时间">
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
      <a-button @click="handleAdd" type="primary" icon="plus">添加报修</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('故障报修')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
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
    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="操作记录" key="1">
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
      <a-tab-pane tab="存档文档" key="2" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
      <a-tab-pane tab="故障图片" key="3" forceRender>
        <Fault-pictures ref="faultPictures"></Fault-pictures>
      </a-tab-pane>
    </a-tabs>

    <trouble-fix-report-modal ref="modalForm" @ok="modalFormOk"></trouble-fix-report-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import OperatingRecordList from './modules/operatingRecordList'
import ArchivedDocumentList from './modules/archivedDocumentList'
import FaultPictures from './modules/faultPictures'
import { queryDepartTreeList } from '@/api/api'

export default {
  name: 'TroubleFixReportList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    OperatingRecordList,
    FaultPictures,
    ArchivedDocumentList
  },
  data() {
    return {
      queryTime: [],
      queryParam: { dealStatus: '' },
      description: '故障报修管理页面',
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
          dataIndex: 'fixNo'
        },
        {
          title: '处理状态',
          align: 'center',
          dataIndex: 'dealStatus'
        },
        // {
        //   title: '设备编码',
        //   align: 'center',
        //   dataIndex: 'equipmentCode'
        // },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'equipmentName'
        },
        {
          title: '报修时间',
          align: 'center',
          dataIndex: 'troubleTime'
        },
        {
          title: '故障描述',
          align: 'center',
          dataIndex: 'troubleDescription'
        },
        // {
        //   title: '设备类型',
        //   align: 'center',
        //   dataIndex: 'equipmentType'
        // },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'equipmentModel'
        },
        {
          title: '设备位置',
          align: 'center',
          dataIndex: 'equipmentAddress'
        },
        // {
        //   title: '故障信息',
        //   align: 'center',
        //   dataIndex: 'troubleMessage'
        // },
        // {
        //   title: '报修人电话',
        //   align: 'center',
        //   dataIndex: 'troubleTel'
        // },
        // {
        //   title: '故障等级',
        //   align: 'center',
        //   dataIndex: 'troubleLevel_dictText'
        // },
        // {
        //   title: '故障类型',
        //   align: 'center',
        //   dataIndex: 'troubleType_dictText'
        // },
        // {
        //   title: '故障时间',
        //   align: 'center',
        //   dataIndex: 'troubleTime'
        // },
        {
          title: '维修组',
          align: 'center',
          dataIndex: 'fixGroup'
        },
        // {
        //   title: '紧急程度',
        //   align: 'center',
        //   dataIndex: 'urgency_dictText'
        // },

        // {
        //   title: '审核人',
        //   align: 'center',
        //   dataIndex: 'examineId'
        // },
        // {
        //   title: '审核意见',
        //   align: 'center',
        //   dataIndex: 'examineSuggest'
        // },

        // {
        //   title: '维修班组',
        //   align: 'center',
        //   dataIndex: 'fixTeam_dictText'
        // },
        // {
        //   title: '维修人员',
        //   align: 'center',
        //   dataIndex: 'fixStaff_dictText'
        // },
        // {
        //   title: '处理意见',
        //   align: 'center',
        //   dataIndex: 'fixSuggest_dictText'
        // },
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
        list: '/troublereport/tabTroubleFixReport/list',
        delete: '/troublereport/tabTroubleFixReport/delete',
        deleteBatch: '/troublereport/tabTroubleFixReport/deleteBatch',
        exportXlsUrl: '/troublereport/tabTroubleFixReport/exportXls',
        importExcelUrl: 'troublereport/tabTroubleFixReport/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        console.log('arr', arr)
        arr.forEach(item => {
          let pm = {
            value: item.id,
            label: item.departName
          }
          this.treeData.push(pm)
        })
        // this.treeData = arr
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.proposedTimeStartTime = dateString[0]
      this.queryParam.proposedTimeEndTime = dateString[1]
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.operatingRecordList.details(record)
            this.$refs.archivedDocumentList.details(record)
            this.$refs.faultPictures.details(record)
            // this.$refs.modalForm.edit(record)
            getAction('/info/tabStock/queryById?id=' + record.id).then(res => {
              if (res.success) {
                this.outList.dataSource = res.result.detailList || []
              }
            })
          },

          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'fixNo', text: '单号' })
      fieldList.push({ type: 'string', value: 'equipmentName', text: '设备名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentModel', text: '规格型号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'address', text: '设备位置', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troubleLevel', text: '故障等级', dictCode: 'failure_level' })
      fieldList.push({ type: 'date', value: '', text: '提出开始时间' })
      fieldList.push({ type: 'date', value: '', text: '提出开结束时间' })
      // fieldList.push({ type: 'string', value: 'fixGroup', text: '维修组', options: this.treeData })
      fieldList.push({ type: 'string', value: 'troubleDescription', text: '故障描述', dictCode: '' })

      this.superFieldList = fieldList
      // fieldList.push({ type: 'string', value: 'troubleTel', text: '报修人电话', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'createBy', text: '报修人', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'urgency', text: '紧急程度', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'equipmentCode', text: '设备编码', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'equipmentType', text: '设备类型', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'troubleMessage', text: '故障信息', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'troubleType', text: '故障类型', dictCode: '' })
      // fieldList.push({ type: 'datetime', value: 'troubleTime', text: '故障时间' })
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
