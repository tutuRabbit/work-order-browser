<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="状态">
              <a-select v-model.trim="queryParam.status" style="width: 100%">
                <a-select-option value="0">外委中</a-select-option>
                <a-select-option value="1">待验证</a-select-option>
                <a-select-option value="2">已完成</a-select-option>
              </a-select>
              <!-- <a-input placeholder="" v-model="queryParam.username"></a-input> -->
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备信息">
              <a-input v-model="queryParam.equipment"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="外委单位">
                <a-input v-model="queryParam.entrustCompany"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="维修单号">
                <a-input v-model="queryParam.fixWorkOrderId"></a-input>
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
      <a-button @click="handleAdd" type="primary" icon="plus">添加申请</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('外委维修')">导出</a-button>
      <a-button type="primary">打印</a-button>
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
          <a v-if="record.status == '0'" @click="handleExecute(record)">执行外委</a>
          <a-divider v-if="record.status == '0'" type="vertical" />
          <a v-if="record.status == '1'" @click="handleVerify(record)">验证</a>
          <a-divider v-if="record.status == '1'" type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
          <a-divider type="vertical"></a-divider>
          <a @click="handleDetail(record)">详情</a>
          <!-- <a @click="handleEdit(record)">编辑</a>
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
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <a-tabs defaultActiveKey="0">
      <a-tab-pane tab="操作记录" key="0" forceRender>
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
      <a-tab-pane tab="存档文档" key="1" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
      <a-tab-pane tab="故障图片" key="2" forceRender>
        <Fault-pictures ref="faultPictures"></Fault-pictures>
      </a-tab-pane>
    </a-tabs>

    <trouble-fix-report-modal ref="modalForm" @ok="modalFormOk"></trouble-fix-report-modal>
    <dispatching-modal-form ref="dispatchingModalForm" @ok="modalFormOk"></dispatching-modal-form>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import dispatchingModalForm from './modules/dispatchingModalForm'
import OperatingRecordList from './modules/operatingRecordList'
import ArchivedDocumentList from './modules/archivedDocumentList'
import FaultPictures from './modules/faultPictures'

export default {
  name: 'TroubleFixReportList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    dispatchingModalForm,
    OperatingRecordList,
    FaultPictures,
    ArchivedDocumentList
  },
  data() {
    return {
      description: '外委维修管理页面',
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
          title: '状态',
          align: 'center',
          dataIndex: 'status',
          customRender: function(t, r, index) {
            if (r.status == '0') {
              return '外委中'
            } else if (r.status == '2') {
              return '待验证'
            } else {
              return '已完成'
            }
          }
        },
        {
          title: '外委单号',
          align: 'center',
          dataIndex: 'outFixEntrustNo'
        },
        {
          title: '设备编码',
          align: 'center',
          dataIndex: 'equipmentCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'equipmentName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'equipmentModel'
        },
        {
          title: '外委单位',
          align: 'center',
          dataIndex: 'entrustCompany'
        },
        {
          title: '实际费用',
          align: 'center',
          dataIndex: 'predictCost'
          // dataIndex: 'tureCost'
        },
        {
          title: '原因',
          align: 'center',
          dataIndex: 'applyReason'
        },
        {
          title: '外委内容',
          align: 'center',
          dataIndex: 'outEntrustContent'
        },
        {
          title: '维修单号',
          align: 'center',
          dataIndex: 'fixWorkOrderNo'
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
        list: '/fixworkorder/outFixEntrust/list',
        delete: '/fixworkorder/outFixEntrust/delete',
        deleteBatch: '/fixworkorder/outFixEntrust/deleteBatch',
        exportXlsUrl: '/fixworkorder/outFixEntrust/exportXls',
        importExcelUrl: 'fixworkorder/outFixEntrust/importExcel'
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
    handleExecute(record) {
      this.$refs.modalForm.execute(record)
      this.$refs.modalForm.title = '执行外委'
      this.$refs.modalForm.disableSubmit = false
    },
    handleVerify(record) {
      this.$refs.modalForm.verify(record)
      this.$refs.modalForm.title = '外委验证'
      this.$refs.modalForm.disableSubmit = false
    },
    handleDispatching() {
      let row = this.selectionRows
      if (row.length == 1 && row[0].fixStatus == 'fixing') {
        console.log('派工')
        this.$refs.dispatchingModalForm.editor()
      } else {
        this.$message.warning('请选择一条维修中的数据!')
      }
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.operatingRecordList.details(record)
            this.$refs.archivedDocumentList.details(record)
            this.$refs.faultPictures.details(record)
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
      fieldList.push({ type: 'string', value: 'repairOrderNo', text: '维修单号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentCode', text: '设备编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentName', text: '设备名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentModel', text: '规格型号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'depart', text: '所在部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentType', text: '设备类别', dictCode: 'B03' })
      fieldList.push({ type: 'datetime', value: 'troubleTime', text: '发生时间' })
      fieldList.push({ type: 'string', value: 'operator', text: '操作人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troublePart', text: '故障部位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troubleType', text: '故障类别', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troubleLevel', text: '故障等级', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troubleDescription', text: '故障描述', dictCode: '' })
      fieldList.push({ type: 'string', value: 'troubleReason', text: '故障原因', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixTeam', text: '维修组', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixStatus', text: '维修状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixLevel', text: '维修级别', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixType', text: '维修类别', dictCode: '' })
      fieldList.push({ type: 'string', value: 'urgency', text: '紧急程度', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stop', text: '是否停机', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stopTime', text: '停机时长', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'startTime', text: '维修开始时间' })
      fieldList.push({ type: 'datetime', value: 'endTime', text: '维修结束时间' })
      fieldList.push({ type: 'string', value: 'fixHour', text: '维修用时（时）', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixMin', text: '维修用时（分）', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixSpend', text: '维修费用', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outCompany', text: '外委单位', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
