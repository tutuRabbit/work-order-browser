<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="计划名称">
              <a-input v-model="queryParam.planName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="计划所属部门">
              <a-tree-select
                v-model.trim="queryParam.planDepartment"
                style="width: 100%"
                :tree-data="treeData"
                placeholder="请选择"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
              >
              </a-tree-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备所属部门">
              <a-tree-select
                v-model.trim="queryParam.sysOrgCode"
                style="width: 100%"
                :tree-data="treeData"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                placeholder="请选择"
              >
              </a-tree-select>
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
      <a-button @click="handleChange" type="primary" icon="plus">变更任务</a-button>
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="import" @click="handleExportXls('巡检任务')">导出</a-button>
      <a-button type="primary">打印</a-button>
      <a-button type="primary" @click="handleJump">跳过巡检</a-button>
      <!-- <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <!-- <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query> -->
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
    </div>
    <div class="table-operator">
      <span style="font-size:16px">【状态】：</span>
      <a-radio-group @change="searchQuery" v-model="queryParam.status" button-style="solid">
        <a-radio-button style="color: #000cff" value="1">执行中(0)</a-radio-button>
        <a-radio-button style="color: #3bd938" value="4">待执行(0)</a-radio-button>
        <a-radio-button style="color: #0049ff" value="5">未开始(0)</a-radio-button>
        <a-radio-button style="color: #ff00a9" value="6">已完成(0)</a-radio-button>
      </a-radio-group>
      <span style="font-size:16px">【设备巡检进度】：</span>
      <a-radio-group v-model="queryParam.isOverdue" button-style="solid" @change="searchQuery">
        <a-radio-button style="color: #000cff" value="1">未检(0)</a-radio-button>
        <a-radio-button style="color: #f00" value="0">异常(0)</a-radio-button>
      </a-radio-group>
      <a-button @click="handleReset" style="margin-left:10px">重置状态</a-button>
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
        :scroll="{ x: true, y: 300 }"
        :bordered="false"
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

        <span slot="action" slot-scope="text, record">
          <a v-if="record.status != '已完成'" @click="handleEdit(record)">执行</a>
        </span>
      </a-table>
    </div>
    <div class="title">设备情况</div>
    <Check-record-List @ok="loadData" ref="checkRecordList"></Check-record-List>
    <patrol-inspection-standard-modal ref="modalForm" @ok="modalFormOk"></patrol-inspection-standard-modal>
    <!-- 变更任务 -->
    <verify-modal-form ref="verifyModalForm"></verify-modal-form>
    <!-- 跳过巡检 -->
    <j-modal title="跳过巡检" width="600px" :visible="jumpVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="跳过原因">
                <a-select
                  v-model.trim="jumpParam.planTime"
                  style="width: 100%"
                  :showSearch="true"
                  optionFilterProp="label"
                  placeholder="请选择"
                >
                  <a-select-option value="0" label="保养中">保养中</a-select-option>
                  <a-select-option value="1" label="外委中">外委中</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="jumpOk">确认</a-button>
      </template>
    </j-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import verifyModalForm from './modules/verifyModalForm'
import CheckRecordList from './modules/checkRecordList'
import { putAction } from '@api/manage'
import { queryDepartTreeList } from '@/api/api'
import PatrolInspectionStandardModal from './modules/PatrolInspectionStandardModal'

export default {
  name: 'PatrolInspectionStandardList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    verifyModalForm,
    CheckRecordList,
    PatrolInspectionStandardModal
  },
  data() {
    return {
      description: '巡检任务管理页面',
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
        // {
        //   title: '计划id',
        //   align: 'center',
        //   dataIndex: 'planId'
        // },
        // {
        //   title: '计划编号',
        //   align: 'center',
        //   dataIndex: 'planNo'
        // },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'status'
        },
        {
          title: '计划名称',
          align: 'center',
          dataIndex: 'planName'
        },
        {
          title: '设备巡检进度',
          align: 'center',
          dataIndex: 'patrolInspectionProgress'
          // customRender: function(t, r, index) {
          //   if (r.patrolInspectionProgress == '0') {
          //     return '全部'
          //   } else if (r.patrolInspectionProgress == '1') {
          //     return '全部未检'
          //   }
          // }
        },
        {
          title: '计划巡检开始时间',
          align: 'center',
          dataIndex: 'planStartTime'
        },
        {
          title: '计划巡检结束时间',
          align: 'center',
          dataIndex: 'planEndTime'
        },
        {
          title: '巡检执行开始时间',
          align: 'center',
          dataIndex: 'checkStartTime'
        },
        {
          title: '巡检执行结束时间',
          align: 'center',
          dataIndex: 'checkEndTime'
        },
        {
          title: '计划巡检人',
          align: 'center',
          dataIndex: 'planExecutor'
        },
        {
          title: '巡检人',
          align: 'center',
          dataIndex: 'executor'
        },
        {
          title: '计划所属部门',
          align: 'center',
          dataIndex: 'planDepartment'
        },
        // {
        //   title: '显示',
        //   align: 'center',
        //   dataIndex: 'displayMark'
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
        list: '/checkplan/patrolInspectionStandard/list',
        delete: '/checkplan/patrolInspectionStandard/delete',
        deleteBatch: '/checkplan/patrolInspectionStandard/deleteBatch',
        exportXlsUrl: '/checkplan/patrolInspectionStandard/exportXls',
        patrolInspectionTask: '/checkplan/patrolInspectionTask', //跳过任务
        importExcelUrl: 'checkplan/patrolInspectionStandard/importExcel'
      },
      dictOptions: {},
      jumpParam: {},
      treeData: [],
      jumpVisible: false,
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handleEdit(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '执行'
      this.$refs.modalForm.disableSubmit = false
    },
    handleJump() {
      if (this.selectedRowKeys.length == 1) {
        this.jumpParam.id = this.selectionRows[0].id
        this.jumpVisible = true
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    handleCancel() {
      this.jumpVisible = false
      this.jumpParam = {}
    },
    jumpOk() {
      let pm = new URLSearchParams()
      pm.append('id', this.jumpParam.id)
      pm.append('skipReason', this.jumpParam.planTime)
      putAction(this.url.patrolInspectionTask, pm).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.$message.success(res.message)
            this.loadData()
          }
        } else {
          this.$message.warning(res.message)
        }
        this.jumpVisible = false
      })
    },
    handleReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleChange() {
      if (this.selectedRowKeys.length == 1) {
        let dataList = this.selectionRows.map(item => item.keepProcessStatus)
        this.$refs.verifyModalForm.editor(this.selectionRows)
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'planId', text: '计划id', dictCode: '' })
      fieldList.push({ type: 'string', value: 'planNo', text: '计划编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'status', text: '状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'planName', text: '计划名称', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'planStartTime', text: '计划巡检开始时间' })
      fieldList.push({ type: 'datetime', value: 'planEndTime', text: '计划巡检结束时间' })
      fieldList.push({ type: 'datetime', value: 'checkStartTime', text: '巡检执行开始时间' })
      fieldList.push({ type: 'datetime', value: 'checkEndTime', text: '巡检执行结束时间' })
      fieldList.push({ type: 'string', value: 'planExecutor', text: '巡检人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'planDepartment', text: '计划所属部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'displayMark', text: '显示', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';

.ant-radio-button-wrapper {
  margin: 0 10px;
}
</style>
