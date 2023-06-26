<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="计划保养时间">
              <j-dict-select-tag
                type="list"
                v-model="queryParam.planTime"
                dictCode="keep_time_type"
                placeholder="请选择计划保养时间"
              />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="保养班组">
              <!-- <j-dict-select-tag
                type="list"
                v-model="queryParam.keepUsers"
                dictCode="maintain_list"
                placeholder="请选择保养班组"
              /> -->
              <a-tree-select
                v-model.trim="queryParam.keepUsers"
                style="width: 100%"
                :tree-data="treeData"
                placeholder="请选择保养班组"
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
                  placeholder="请选择使用部门"
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
      <a-button @click="handleAdd" type="primary" icon="plus">添加计划</a-button>
      <a-button @click="handleBlockUp('1')" type="primary">启用</a-button>
      <a-button @click="handleBlockUp('0')" type="primary">停用</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('保养计划表')">导出</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <!-- <a-menu-item key="1" @click="batchDel">派工</a-menu-item> -->
          <a-menu-item key="1" @click="batchDel">删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
    </div>
    <!-- <div class="table-operator"> -->
    <div style="margin-bottom:10px">
      <span style="font-size:16px">【状态】：</span>
      <a-radio-group @change="searchQuery" v-model="queryParam.keepProcessStatus" button-style="solid">
        <a-radio-button style="color: #008fca;" value="1">未开始({{ this.status1Num }})</a-radio-button>
        <a-radio-button style="color: #ff9090" value="2">待接单/派工({{ this.status2Num }})</a-radio-button>
        <a-radio-button style="color: #ffc107" value="3">待保养({{ this.status3Num }})</a-radio-button>
        <a-radio-button style="color: #1d9c61" value="4">保养中({{ this.status4Num }})</a-radio-button>
        <a-radio-button style="color: #0049ff" value="5">待验证({{ this.status5Num }})</a-radio-button>
      </a-radio-group>
      <span style="font-size:16px">【逾期情况】：</span>
      <a-radio-group v-model="queryParam.isOverdue" button-style="solid" @change="searchQuery">
        <a-radio-button style="color: #f00" value="1">计划已逾期({{ this.overStatusNum }})</a-radio-button>
      </a-radio-group>
      <span style="font-size:16px">【停用状态】：</span>
      <a-radio-group v-model="queryParam.enableStatus" button-style="solid" @change="searchQuery">
        <a-radio-button style="color: #f00" value="0">停用({{ this.stopStatusNum }})</a-radio-button>
      </a-radio-group>
      <!-- <a-button @click="handleReset">重置状态</a-button> -->
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
            @click="handleDispatching(record)"
            v-if="record.keepProcessStatus == '待派工' || record.keepProcessStatus == '未开始'"
            >派工</a
          >
          <a-divider
            v-if="record.keepProcessStatus == '待派工' || record.keepProcessStatus == '未开始'"
            type="vertical"
          />
          <a
            v-if="record.keepProcessStatus == '待派工' || record.keepProcessStatus == '未开始'"
            @click="handleExecute(record)"
            >执行计划</a
          >
          <a-divider
            v-if="record.keepProcessStatus == '待派工' || record.keepProcessStatus == '未开始'"
            type="vertical"
          />
          <!-- <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a @click="handleDetail(record)">详情</a>
          <a-divider type="vertical" /> -->
          <a @click="handleDelete(record.id)">删除</a>
        </span>
      </a-table>
    </div>

    <trouble-fix-report-modal ref="modalForm" @ok="modalFormOk"></trouble-fix-report-modal>
    <dispatching-modal-form ref="dispatchingModalForm" @ok="modalFormOk"></dispatching-modal-form>
    <Advanced-query ref="advancedQuery"></Advanced-query>
    <!-- 高级查询 -->
    <j-modal title="更多查询" width="50%" :visible="visible" @cancel="handleCancel" cancelText="关闭">
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
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { queryDepartTreeList } from '@/api/api'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TroubleFixReportModal from './modules/TroubleFixReportModal'
import dispatchingModalForm from './modules/dispatchingModalForm'
import AdvancedQuery from './modules/advancedQuery'
import { putAction, getAction } from '@/api/manage'
import moment from 'dayjs'

export default {
  name: 'TabKeepOrderList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TroubleFixReportModal,
    AdvancedQuery,
    dispatchingModalForm
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
      visible: false,
      description: '保养计划管理页面',
      treeData: [],
      dictOptions: [
        {
          label: '未开始',
          value: '1'
        },
        {
          label: '待派工',
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
        {
          title: '保养组',
          align: 'center',
          dataIndex: 'keepUsers'
        },
        {
          title: '保养负责人',
          align: 'center',
          dataIndex: 'keepManager'
        },
        {
          title: '部门',
          align: 'center',
          dataIndex: 'sysOrgCode'
        },
        {
          title: '上次保养时间',
          align: 'center',
          dataIndex: 'lastKeepTime'
        },
        {
          title: '下次保养时间',
          align: 'center',
          dataIndex: 'nextKeepTime'
        },
        {
          title: '循环周期',
          align: 'center',
          dataIndex: 'circlePeriod'
        },
        // {
        //   title: '保养类型',
        //   align: 'center',
        //   dataIndex: 'keepType'
        // },
        //保养类型： 0：工单1：计划
        // {
        //   title: '计划保养时间计算规则',
        //   align: 'center',
        //   dataIndex: '22'
        // },
        {
          title: '计划描述',
          align: 'center',
          dataIndex: 'describeText'
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
        list: '/keepOrder/tabKeepOrder/getKeepPlan',
        delete: '/keepOrder/tabKeepOrder/delete',
        deleteBatch: '/keepOrder/tabKeepOrder/deleteBatch',
        exportXlsUrl: '/keepOrder/tabKeepOrder/exportXls',
        modifyStatus: '/keepOrder/tabKeepOrder/modifyStatus', //启用停用
        importExcelUrl: 'keepOrder/tabKeepOrder/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    // this.$route.query.index
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  watch: {
    $route: {
      immediate: true,
      handler() {
        // console.log('============= 哒哒哒s ============= ')
        this.queryParam.id = this.$route.query.index
        this.searchQuery()
        // console.log('$route 哒哒哒s: ',  this.queryParam.id )
      }
    }
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
            //update-begin---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
            //update-end---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
      // 保养计划 状态个数
      getAction('/keepOrder/tabKeepOrder/GetMaintainPlanStatus').then(res => {
        let data = res.result
        console.log('保养计划', data)
        // 状态 status1:未开始 status2:待派工 status3:待保养
        //  status4:保养中 status5:待验证 overStatus:逾期 stopStatus:停用
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
    handleCancel() {
      this.visible = false
    },
    handleReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    handleOk() {
      this.loadData(1)
      // 点击查询清空列表选中行
      // https://gitee.com/jeecg/jeecg-boot/issues/I4KTU1
      this.selectedRowKeys = []
      this.selectionRows = []
      this.visible = false
    },
    // editor
    handleOpen() {
      console.log('111')
      this.$refs.advancedQuery.editor()
    },
    handleBlockUp(val) {
      console.log('val', val)
      console.log('this.selectionRows[0].enableStatus', this.selectionRows[0].enableStatus)
      if (this.selectedRowKeys.length == 1) {
        if (this.selectionRows[0].enableStatus != val) {
          var id = this.selectedRowKeys.map(item => item)
          var pm = new URLSearchParams()
          if (val == '0') {
            // 停用
            pm.append('id', id)
            pm.append('status', '0')
          } else {
            // 启用
            pm.append('id', id)
            pm.append('status', '1')
          }
          putAction(this.url.modifyStatus, pm).then(res => {
            if (res.success) {
              this.$message.success(res.message)
            } else {
              this.$message.warning(res.message)
            }
            this.loadData()
          })
        } else {
          this.$message.warning('计划状态不符,请重试！')
        }
      } else {
        this.$message.warning('请选中一条数据！')
      }
      // if (val == 0) {
      //   let pm = new URLSearchParams()
      //   pm.append('id',id)
      //   pm.append('status', '0')
      //   putAction(this.url.modifyStatus, pm).then(res => {
      //     if (res.success) {
      //       this.$message.success(res.message)
      //     } else {
      //       this.$message.warning(res.message)
      //     }
      //     this.loadData()
      //   })
      // }
    },
    handleDispatching(record) {
      this.$refs.modalForm.dispatching(record)
      this.$refs.modalForm.title = '派工'
      this.$refs.modalForm.disableSubmit = true
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            // this.$refs.archivedDocumentList.details(record)
            // this.$refs.faultPictures.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    handleExecute(record) {
      this.$refs.modalForm.execute(record)
      this.$refs.modalForm.title = '执行计划'
      this.$refs.modalForm.disableSubmit = false
    },
    initDictConfig() {}
  }
}
</script>
<style lang="less" scoped>
.ant-radio-button-wrapper {
  margin: 0 10px;
}
.beginA :hover {
  color: #000;
}
.beginA :active {
  color: #000;
}
.ant-radio-group-solid .ant-radio-button-wrapper-checked:not(.ant-radio-button-wrapper-disabled) {
  color: #fff;
  /* background: #ebf5ff; */
  border-color: #1890ff;
}
</style>
