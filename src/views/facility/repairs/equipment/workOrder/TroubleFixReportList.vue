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
      <a-button @click="handleAdd" type="primary" icon="plus">快速工单</a-button>
      <a-button type="primary">打印</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('维修工单')">导出</a-button>
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
          <a
            v-if="record.fixStatus == '维修中' || record.fixStatus == '待停机维修' || record.fixStatus == '待维修'"
            @click="handleContractor(record)"
            >执行维修</a
          >
          <a-divider
            v-if="record.fixStatus == '维修中' || record.fixStatus == '待停机维修' || record.fixStatus == '待维修'"
            type="vertical"
          />
          <a v-if="record.fixStatus == '待停机维修' || record.fixStatus == '待维修'" @click="handleDispatching(record)"
            >派工</a
          >
          <a-divider v-if="record.fixStatus == '待停机维修' || record.fixStatus == '待维修'" type="vertical" />
          <a v-if="record.fixStatus == '待验证'" @click="handleVerify(record)">验证</a>
          <a-divider v-if="record.fixStatus == '待验证'" type="vertical" />
          <a v-if="record.fixStatus == '已完成'" @click="handleGradeForm(record)">评分</a>
          <a-divider v-if="record.fixStatus == '已完成'" type="vertical" />
          <a v-if="record.fixStatus == '待停机维修' || record.fixStatus == '待维修'" @click="handleCommission(record)"
            >申请外委</a
          >
          <a-divider v-if="record.fixStatus == '待停机维修' || record.fixStatus == '待维修'" type="vertical" />
          <a @click="handleDetail(record)">详情</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
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
    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="存档文档" key="1" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
      <a-tab-pane tab="故障图片" key="2" forceRender>
        <Fault-pictures ref="faultPictures"></Fault-pictures>
      </a-tab-pane>
    </a-tabs>

    <trouble-fix-report-modal ref="modalForm" @ok="loadData"></trouble-fix-report-modal>
    <!-- 派工 -->
    <dispatching-modal-form ref="dispatchingModalForm" @ok="loadData"></dispatching-modal-form>
    <!-- 申请外委 -->
    <commission-modal-form ref="commissionModalForm" @ok="modalFormOk"></commission-modal-form>
    <!-- 评分 -->
    <j-modal
      title="评分"
      :width="width"
      :visible="gradeVisible"
      :okButtonProps="{ class: { 'jee-hidden': true } }"
      @cancel="handleGradeCancel"
      cancelText="关闭"
    >
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <div class="title" style="margin-top:4px">注：本页为评分查看页,评价请执行验证操作！</div></a-col
            >
            <a-col :span="24">
              <a-form-item label="评分">
                <a-rate :disabled="true" v-model="gradeParam.score" />
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="评价">
                <a-input :disabled="true" v-model="gradeParam.evaluate" type="textarea"></a-input>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
    <!-- 验证 -->
    <j-modal
      title="验证"
      :fullscreen="true"
      :visible="verifyVisible"
      @ok="handleVerifyOk"
      @cancel="handleGradeCancel"
      cancelText="关闭"
    >
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-table
                ref="table"
                size="middle"
                bordered
                rowKey="id"
                :height="150"
                :disabled="true"
                :columns="verifyTable.columns"
                :dataSource="verifyTable.dataSource"
              >
              </a-table>
            </a-col>
            <a-col :span="24"><div class="title" style="margin-top:20px">验证</div></a-col>
            <a-col :span="24">
              <a-form-item label="是否修复">
                <a-radio-group v-model="verifyParam.repairOk">
                  <a-radio value="0">是</a-radio>
                  <a-radio value="1">否</a-radio>
                </a-radio-group>
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="重新维修">
                <a-radio-group v-model="verifyParam.reRepair" :disabled="verifyParam.repairOk == '0'">
                  <a-radio value="0">是</a-radio>
                  <a-radio value="1">否</a-radio>
                </a-radio-group>
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="评分">
                <a-rate v-model="verifyParam.score" />
              </a-form-item>
            </a-col>
            <a-col :span="18">
              <a-form-item label="检查人">
                <j-select-multi-user v-model="verifyParam.inspectedBy"></j-select-multi-user>
              </a-form-item>
            </a-col>
            <a-col :span="18">
              <a-form-item label="评价">
                <j-dict-select-tag
                  v-model="verifyParam.evaluate"
                  placeholder="请选择评价"
                  dictCode="the_guy_single_evaluate"
                />
                <!-- <a-input v-model="verifyParam.evaluate" type="textarea"></a-input> -->
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </j-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { getAction } from '@/api/manage'
import { mixinDevice } from '@/utils/mixin'
import { putAction } from '@api/manage'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import dispatchingModalForm from './modules/dispatchingModalForm'
import commissionModalForm from './modules/commissionModalForm'
import ArchivedDocumentList from './modules/archivedDocumentList'
import FaultPictures from './modules/faultPictures'

export default {
  name: 'TroubleFixReportList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    dispatchingModalForm,
    commissionModalForm,
    FaultPictures,
    ArchivedDocumentList
  },
  data() {
    return {
      verifyTable: {
        dataSource: [],
        columns: [
          {
            title: '维修单号',
            align: 'center',
            dataIndex: 'repairOrderNo'
          },
          {
            title: '设备编号',
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
            title: '使用部门',
            align: 'center',
            dataIndex: 'depart'
          },
          {
            title: '故障原因',
            align: 'center',
            dataIndex: 'troubleReason'
          }
        ]
      },
      description: '故障报修管理页面',
      gradeVisible: false,
      verifyVisible: false,
      width: 600,
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
          title: '维修单号',
          align: 'center',
          dataIndex: 'repairOrderNo'
        },
        {
          title: '处理状态',
          align: 'center',
          dataIndex: 'fixStatus'
        },
        {
          title: '维修组',
          align: 'center',
          dataIndex: 'fixTeam'
        },
        {
          title: '主修人员',
          align: 'center',
          dataIndex: 'mainFix'
        },
        {
          title: '维修开始时间',
          align: 'center',
          dataIndex: 'startTime'
        },
        {
          title: '维修结束时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'equipmentName'
        },
        {
          title: '使用部门',
          align: 'center',
          dataIndex: 'depart'
        },
        {
          title: '故障描述',
          align: 'center',
          dataIndex: 'troubleDescription'
        },
        {
          title: '报修人',
          align: 'center',
          dataIndex: 'createBy'
        },
        {
          title: '报修时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '维修用时(小时)',
          align: 'center',
          dataIndex: 'fixHour'
        },
        {
          title: '辅修人员',
          align: 'center'
          // dataIndex: 'troubleTime'
        },
        {
          title: '设备编号',
          align: 'center',
          dataIndex: 'equipmentCode'
        },

        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'equipmentModel'
        },
        {
          title: '维修工时',
          align: 'center'
          // dataIndex: 'examineSuggest'
        },

        {
          title: '停机时间(小时)',
          align: 'center',
          dataIndex: 'stopTime'
        },
        {
          title: '故障类别',
          align: 'center',
          dataIndex: 'troubleType'
        },
        {
          title: '维修类型',
          align: 'center',
          dataIndex: 'fixType'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 200,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/fixworkorder/fixWorkOrder/list',
        delete: '/fixworkorder/fixWorkOrder/delete',
        deleteBatch: '/fixworkorder/fixWorkOrder/deleteBatch',
        exportXlsUrl: '/fixworkorder/fixWorkOrder/exportXls'
      },
      gradeParam: {},
      verifyParam: {
        repairOk: '0',
        reRepair: '1',
        score: 5
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
    handleGradeOk() {
      var pm = new URLSearchParams()
      pm.append('id', this.gradeParam.id)
      pm.append('score', this.gradeParam.score)
      putAction('/fixworkorder/fixWorkOrder/score', pm).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
            this.handleGradeCancel()
            this.loadData()
          }
        } else {
          this.$message.warning(res.message)
        }
        this.visible = false
      })
    },
    handleVerifyOk() {
      // var pm = new URLSearchParams()
      // pm.append('id', this.verifyParam.id)
      // pm.append('score', this.verifyParam.score)
      putAction('/fixworkorder/fixWorkOrder/verification', this.verifyParam).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
            this.handleGradeCancel()
            this.loadData()
          }
        } else {
          this.$message.warning(res.message)
        }
        this.visible = false
      })
    },
    handleGradeCancel() {
      this.gradeVisible = false
      this.gradeParam = {}
      this.verifyVisible = false
      this.verifyParam = {}
    },
    handleDispatching(record) {
      // let row = this.selectionRows
      // if (row.length == 1 && (row[0].fixStatus == '维修中' || row[0].fixStatus == '待停机维修')) {
      //   console.log('派工')
      //   this.$refs.dispatchingModalForm.editor(row)
      // } else {
      //   this.$message.warning('请选择一条维修中的数据!')
      // }
      this.$refs.dispatchingModalForm.editor(record)
    },
    handleCommission(record) {
      this.$refs.commissionModalForm.editor(record)
    },
    // 执行维修
    handleContractor: function(record) {
      this.$refs.modalForm.execute(record)
      this.$refs.modalForm.title = '执行维修'
      this.$refs.modalForm.disableSubmit = false
    },
    handleGradeForm(record) {
      console.log('record', record)
      console.log('record', record.score)
      console.log('record', record.evaluate)
      // this.gradeParam.id = record.id
      this.gradeParam.score = record.score
      this.gradeParam.evaluate = record.evaluate
      console.log('this.gradeParam', this.gradeParam)
      this.gradeVisible = true
    },
    handleVerify(record) {
      // console.log('record', record)
      this.verifyTable.dataSource = []
      let pm = {
        repairOrderNo: record.repairOrderNo,
        equipmentCode: record.equipmentCode,
        equipmentName: record.equipmentName,
        equipmentModel: record.equipmentModel,
        depart: record.depart,
        troubleReason: record.troubleReason
      }
      this.verifyTable.dataSource.push(pm)
      this.verifyVisible = true
      this.verifyParam.id = record.id
      console.log('this.verifyParam', this.verifyParam)
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
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
