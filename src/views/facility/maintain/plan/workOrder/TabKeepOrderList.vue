<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="计划保养时间">
              <a-select
                v-model.trim="queryParam.planTime"
                style="width: 100%"
                placeholder="请选择"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option value="" label="全部">全部</a-select-option>
                <a-select-option value="0" label="今日">今日</a-select-option>
                <a-select-option value="1" label="本周">本周</a-select-option>
                <a-select-option value="2" label="本月">本月</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="保养班组">
              <a-tree-select
                v-model.trim="queryParam.keepUsers"
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
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="使用部门">
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
      <a-button @click="handleAdd" type="primary" icon="plus">快速工单</a-button>
      <a-button type="primary" @click="handleVerify">验证</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('保养工单')">导出</a-button>
      <a-button type="primary">打印</a-button>
      <!-- <a-button type="primary" icon="filter" @click="visible = true">高级查询</a-button> -->
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
    </div>
    <div class="table-operator">
      <span style="font-size:16px">【状态】：</span>
      <a-radio-group @change="searchQuery" v-model="queryParam.keepProcessStatus" button-style="solid">
        <a-radio-button style="color: #008fca" value="3">待保养({{ this.status3Num }})</a-radio-button>
        <a-radio-button style="color: #1d9c61" value="4">保养中({{ this.status4Num }})</a-radio-button>
        <a-radio-button style="color: #0049ff" value="5">待验证({{ this.status5Num }})</a-radio-button>
      </a-radio-group>
      <span style="font-size:16px">【逾期情况】：</span>
      <a-radio-group v-model="queryParam.isOverdue" button-style="solid" @change="searchQuery">
        <a-radio-button style="color: #f00" value="1">计划已逾期({{ this.overStatusNum }})</a-radio-button>
      </a-radio-group>
      <!-- <a-button @click="handleReset" style="margin-left:10px">重置状态</a-button> -->
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
            v-if="record.keepProcessStatus == '待保养' || record.keepProcessStatus == '保养中'"
            @click="handleExecute(record)"
            >执行保养</a
          >
          <a-divider
            v-if="record.keepProcessStatus == '待保养' || record.keepProcessStatus == '保养中'"
            type="vertical"
          />
          <!-- <a v-if="record.keepProcessStatus == '待验证'" @click="handleExecute(record)">验证</a>
          <a-divider v-if="record.keepProcessStatus == '待验证'" type="vertical" /> -->
          <a v-if="record.keepProcessStatus == '已完成'" @click="handleScore(record)">评分</a>
          <a-divider v-if="record.keepProcessStatus == '已完成'" type="vertical" />
          <a @click="handleDetail(record)">详情</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>
        <span slot="keepUseTime" slot-scope="text, record">
          <span>{{ record.keepHour ? record.keepHour : 0 }}小时{{ record.keepMin ? record.keepMin : 0 }}分</span>
        </span>
      </a-table>
    </div>
    <a-tabs defaultActiveKey="0">
      <a-tab-pane tab="保养明细" key="0" forceRender>
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
      <a-tab-pane tab="存档文档" key="1" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
    </a-tabs>
    <trouble-fix-report-modal ref="modalForm" @ok="modalFormOk"></trouble-fix-report-modal>
    <!-- 高级查询 -->
    <j-modal title="更多查询" width="50%" :visible="visible" @cancel="visible = false" cancelText="关闭">
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="12">
              <a-form-item label="设备编号">
                <a-input v-model="queryParam.deviceCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备名称">
                <a-input v-model="queryParam.deviceName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="规格型号">
                <a-input v-model="queryParam.deviceModel"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.deviceType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="保养级别">
                <j-dict-select-tag
                  type="list"
                  dictCode="keep_level"
                  v-model.trim="queryParam.keepLevel"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="保养负责人">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="queryParam.keepManager"
                  dictCode="sys_user,realname,id"
                ></j-dict-select-tag>
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
    <!-- 评分 -->
    <j-modal title="评价查看" width="45%" :visible="scoreVisible" cancelText="关闭" @cancel="scoreVisible = false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="评分">
                <a-rate v-model="scoreParam.score" disabled />
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="评价">
                <a-input type="textarea" :disabled="true" v-model="scoreParam.evaluate"></a-input>
                <!-- <a-textarea :disabled="true" v-model="scoreParam.deviceName" :auto-size="{ minRows: 2, maxRows: 5 }" /> -->
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button :loading="loading" @click="scoreVisible = false">关闭</a-button>
      </template>
    </j-modal>
    <!-- 验证 -->
    <verify-modal-form ref="verifyModalForm" @ok="modalFormOk"></verify-modal-form>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { queryDepartTreeList } from '@/api/api'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import OperatingRecordList from './modules/operatingRecordList'
import ArchivedDocumentList from './modules/archivedDocumentList'
import verifyModalForm from './modules/verifyModalForm'
import { putAction, getAction } from '@/api/manage'

export default {
  name: 'TabKeepOrderList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    OperatingRecordList,
    verifyModalForm,
    ArchivedDocumentList
  },
  data() {
    return {
      status1Num: 0,
      status2Num: 0,
      status3Num: 0,
      status4Num: 0,
      status5Num: 0,
      overStatusNum: 0,
      stopStatusNum: 0,
      queryParam: {
        planTime: ''
      },
      visible: false,
      scoreVisible: false,
      description: '保养工单表管理页面',
      treeData: [],
      dictOptions: [
        {
          label: '保养中',
          value: '1'
        },
        {
          label: '待保养',
          value: '2'
        },
        {
          label: '待保养',
          value: '3'
        },
        {
          label: '保养中',
          value: '4'
        },
        {
          label: '待验证',
          value: '5'
        },
        {
          label: '已完成',
          value: '6'
        },
        {
          label: '已逾期',
          value: '7'
        }
      ],
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
          title: '保养单号',
          align: 'center',
          dataIndex: 'keepCode'
        },
        {
          title: '保养状态',
          align: 'center',
          dataIndex: 'keepProcessStatus'
        },
        {
          title: '设备编号',
          align: 'center',
          dataIndex: 'deviceCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'deviceName'
        },
        // {
        //   title: '计划序号',
        //   align: 'center',
        //   dataIndex: 'deviceName'
        // },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'deviceModel'
        },
        {
          title: '使用部门',
          align: 'center',
          dataIndex: 'sysOrgCode'
        },
        {
          title: '保养开始时间',
          align: 'center',
          dataIndex: 'startTime',
          dataIndex: 'startTime'
        },
        {
          title: '保养完成时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '保养用时',
          align: 'center',
          dataIndex: 'keepUseTime',
          scopedSlots: { customRender: 'keepUseTime' }
        },
        {
          title: '保养费用',
          align: 'center',
          dataIndex: 'keepPrice'
        },
        // {
        //   title: '备件费用',
        //   align: 'center'
        // dataIndex: 'describeText'
        // },
        // {
        //   title: '保养组',
        //   align: 'center',
        //   dataIndex: 'keepUsers'
        // },
        // {
        //   title: '保养人员',
        //   align: 'center',
        //   dataIndex: 'keepManager'
        // },
        // {
        //   title: '保养信息',
        //   align: 'center',
        //   dataIndex: 'keepManager'
        // },
        // {
        //   title: '验证人',
        //   align: 'center',
        //   dataIndex: 'keepManager'
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
      ipagination: {
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      url: {
        list: '/realKeepOrder/tabKeepOrderReal/list',
        delete: '/realKeepOrder/tabKeepOrderReal/delete',
        deleteBatch: '/realKeepOrder/tabKeepOrderReal/deleteBatch',
        exportXlsUrl: '/realKeepOrder/tabKeepOrderReal/exportXls',
        importExcelUrl: 'realKeepOrder/tabKeepOrderReal/importExcel'
      },
      dictOptions: {},
      scoreParam: {},
      superFieldList: []
    }
  },
  created() {
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
    loadData(arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction(this.url.list, params)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
      // 保养工单 状态个数
      getAction('/realKeepOrder/tabKeepOrderReal/GetMaintainPlanStatus').then(res => {
        let data = res.result
        // 状态 status1:未开始 status2:待派工status3: 待保养
        // status4:保养中 status5:待验证overStatus:逾期 stopStatus:停用
        data.forEach(item => {
          if (item.name == 'status1') {
            this.status1Num = item.num
          }
          if (item.name == 'status2') {
            this.status2Num = item.num
          }
          if (item.name == 'status3') {
            this.status3Num = item.num
          }
          if (item.name == 'status4') {
            this.status4Num = item.num
          }
          if (item.name == 'status5') {
            this.status5Num = item.num
          }
          if (item.name == 'overStatus') {
            this.overStatusNum = item.num
          }
          if (item.name == 'stopStatus') {
            this.stopStatusNum = item.num
          }
        })
      })
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
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.operatingRecordList.details(record)
            this.$refs.archivedDocumentList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    // 执行保养
    handleExecute(record) {
      this.$refs.modalForm.execute(record)
      this.$refs.modalForm.title = '执行保养'
      this.$refs.modalForm.disableSubmit = false
    },
    handleScore(record) {
      console.log('record', record)
      // this.scoreParam = Object.assign({}, record)
      getAction('/keepCheck/tabCheckKeepOrder/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.scoreParam = res.result
          this.$forceUpdate()
        } else {
          this.$message.warning(res.message)
        }
      })
      this.scoreVisible = true
    },
    handleVerify() {
      if (this.selectedRowKeys.length == 0) {
        this.$message.warning('请选中一条数据！')
      } else {
        let dataList = this.selectionRows.map(item => item.keepProcessStatus)
        // let dataId = this.selectionRows.map(item => item.id)
        let verif = '待验证'
        let verifT = dataList.every(item => item === verif)
        if (verifT == true) {
          // console.log('数据全等')
          this.$refs.verifyModalForm.editor(this.selectionRows)
        } else {
          this.$message.warning('保养单状态不符！')
        }
        // httpAction(httpUrl, '', method).then(res => {
        //   if (res.success) {
        //     //重新计算分页问题
        //     this.reCalculatePage(this.selectedRowKeys.length)
        //     this.$message.success(res.message)
        //     this.loadData()
        //     this.onClearSelected()
        //   } else {
        //     this.$message.warning(res.message)
        //   }
        // })
        // this.$message.warning('保养单状态不符！')
      }
    },
    initDictConfig() {}
  }
}
</script>
<style lang="less" scoped>
.ant-radio-button-wrapper {
  margin: 0 10px;
}
</style>
