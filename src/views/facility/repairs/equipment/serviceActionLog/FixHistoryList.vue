<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="8" :sm="12">
            <a-form-item label="维修时间">
              <a-range-picker
                v-model="queryParam.createTimeRange"
                format="YYYY-MM-DD HH:mm"
                :placeholder="['开始时间', '结束时间']"
                @change="onDateChange"
                @ok="onDateOk"
              />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="维修单号">
              <a-input v-model="queryParam.fixOrderNo" placeholder="请输入"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="queryParam.useDepart"
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
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="visible = true" icon="filter" style="margin-left: 8px">更多</a-button>
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
      <a-button v-if="selectedRowKeys.length > 0" type="primary" icon="download" @click="batchDel">删除</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('维修记录')">导出</a-button>
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
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleDetail(record)">详情</a>
        </span>
      </a-table>
    </div>
    <fix-history-modal ref="modalForm" @ok="modalFormOk"></fix-history-modal>
    <!-- 高级查询 -->
    <j-modal title="更多查询" width="50%" :visible="visible" @cancel="visible = false" cancelText="关闭">
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="12">
              <a-form-item label="设备编号">
                <a-input v-model="queryParam.equipmentNo"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备名称">
                <a-input v-model="queryParam.equipmentName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="规格型号">
                <a-input v-model="queryParam.equipmentModel"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.equipmentType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="申请人">
                <a-input v-model="queryParam.applicant"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="维修级别">
                <j-dict-select-tag
                  type="list"
                  dictCode="keep_level"
                  v-model.trim="queryParam.fixLevel"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="维修人员">
                <a-input v-model="queryParam.fixWorker"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="维修费用">
                <a-select style="width:48%;margin-right:10px" v-model="queryParam.fixCost">
                  <a-select-option value="0">大于</a-select-option>
                  <a-select-option value="1">小于</a-select-option>
                  <a-select-option value="2">等于</a-select-option>
                </a-select>
                <a-input style="width:48%" v-model="queryParam.fixCost"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="维修组">
                <j-dict-select-tag
                  type="list"
                  v-model="queryParam.fixGroup"
                  dictCode="maintain_list"
                  placeholder="请选择维修组"
                />
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="维修类别">
                <a-input v-model="queryParam.fixWorker"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="故障原因">
                <j-dict-select-tag
                  type="list"
                  v-model="queryParam.fixWorker"
                  dictCode="failure_cause"
                  placeholder="请选择故障原因"
                />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <div style="float: left">
          <a-button :loading="loading" @click="handleReset">重置</a-button>
        </div>
        <a-button :loading="loading" @click="visible = false">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="handleOk">查询</a-button>
      </template>
    </j-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { queryDepartTreeList } from '@/api/api'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import FixHistoryModal from './modules/FixHistoryModal'

export default {
  name: 'FixHistoryList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    FixHistoryModal
  },
  data() {
    return {
      description: '维修记录管理页面',
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
          // title: '维修单号（维修工单/外委维修）',
          align: 'center',
          dataIndex: 'fixOrderNo'
        },
        {
          title: '设备编码',
          align: 'center',
          dataIndex: 'equipmentNo'
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
          dataIndex: 'useDepart'
        },
        {
          title: '设备类型',
          align: 'center',
          dataIndex: 'equipmentType'
        },
        {
          title: '申请人',
          align: 'center',
          dataIndex: 'applicant'
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
          title: '维修用时（小时）',
          align: 'center',
          dataIndex: 'spendTime'
        },
        {
          title: '停机时长（小时）',
          align: 'center',
          dataIndex: 'stopTime'
        },
        {
          title: '备件费用',
          align: 'center',
          dataIndex: 'spareCost'
        },
        {
          title: '维修级别',
          align: 'center',
          dataIndex: 'fixLevel'
        },
        {
          title: '故障原因',
          align: 'center',
          dataIndex: 'errorReason'
        },
        {
          title: '维修组',
          align: 'center',
          dataIndex: 'fixGroup'
        },
        {
          title: '维修人员',
          align: 'center',
          dataIndex: 'fixWorker'
        },
        {
          title: '验证人',
          align: 'center',
          dataIndex: 'verifiedBy'
        },
        {
          title: '评分',
          align: 'center',
          dataIndex: 'score'
        },
        {
          title: '维修评价',
          align: 'center',
          dataIndex: 'evaluate'
        },
        {
          title: '外委单位',
          align: 'center',
          dataIndex: 'outEntrustCompany'
        },
        {
          title: '维修费用',
          align: 'center',
          dataIndex: 'fixCost'
        },
        {
          title: '故障描述',
          align: 'center',
          dataIndex: 'errorDescription'
        },
        {
          title: '工作描述',
          align: 'center',
          dataIndex: 'workDescription'
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
        list: '/fixHistory/fixHistory/list',
        delete: '/fixHistory/fixHistory/delete',
        deleteBatch: '/fixHistory/fixHistory/deleteBatch',
        exportXlsUrl: '/fixHistory/fixHistory/exportXls',
        importExcelUrl: 'fixHistory/fixHistory/importExcel'
      },
      dictOptions: {},
      queryParam: {},
      treeData: [],
      visible: false,
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
    changeTime(val) {
      console.log('val', val)
      console.log('this.queryParam.planTime', this.queryParam.planTime)
    },
    onDateChange: function(value, dateString) {
      console.log(dateString[0], dateString[1])
      this.queryParam.startTime = dateString[0]
      this.queryParam.endTime = dateString[1]
    },
    onDateOk(value) {
      console.log(value)
    },
    onTimeChange(range) {
      let startTime = range[0]
      let endTime = range[1]
      console.log('endTime', endTime)
      console.log('startTime', startTime)
      // this.startTime = range[0];
      // this.endTime = range[1];
    },
    handleReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleOk() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
      this.visible = false
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'fixOrderNo', text: '维修单号（维修工单/外委维修）', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentNo', text: '设备编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentName', text: '设备名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentModel', text: '规格型号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'useDepart', text: '使用部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'equipmentType', text: '设备类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'applicant', text: '申请人', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'startTime', text: '维修开始时间' })
      fieldList.push({ type: 'datetime', value: 'endTime', text: '维修结束时间' })
      fieldList.push({ type: 'string', value: 'spendTime', text: '维修用时（小时）', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stopTime', text: '停机时长（小时）', dictCode: '' })
      fieldList.push({ type: 'string', value: 'spareCost', text: '备件费用', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixLevel', text: '维修级别', dictCode: '' })
      fieldList.push({ type: 'string', value: 'errorReason', text: '故障原因', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixGroup', text: '维修组', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixWorker', text: '维修人员', dictCode: '' })
      fieldList.push({ type: 'string', value: 'verifiedBy', text: '验证人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'score', text: '评分', dictCode: '' })
      fieldList.push({ type: 'string', value: 'evaluate', text: '维修评价', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outEntrustCompany', text: '外委单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fixCost', text: '维修费用', dictCode: '' })
      fieldList.push({ type: 'string', value: 'errorDescription', text: '故障描述', dictCode: '' })
      fieldList.push({ type: 'string', value: 'workDescription', text: '工作描述', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
