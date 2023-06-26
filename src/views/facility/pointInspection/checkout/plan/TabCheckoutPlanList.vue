<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="8" :sm="12">
            <a-form-item label="检验时间">
              <a-range-picker
                v-model="queryTime"
                :show-time="{ format: 'HH:mm' }"
                @change="onChangeRangeDate"
                format="YYYY-MM-DD HH:mm:ss"
              />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备编号">
              <a-input v-model="queryParam.deviceCode" placeholder="请输入" />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="设备名称">
                <a-input v-model="queryParam.deviceName" placeholder="请输入" />
              </a-form-item>
            </a-col>
            <!-- <a-col :md="6" :sm="8">
              <a-form-item label="是否超期">
                <a-select v-model.trim="queryParam.planTime" style="width: 100%" placeholder="请选择">
                  <a-select-option value="0">未逾期</a-select-option>
                  <a-select-option value="1">已逾期</a-select-option>
                </a-select>
              </a-form-item>
            </a-col> -->
            <a-col :md="6" :sm="8">
              <a-form-item label="检验方式">
                <a-select
                  v-model.trim="queryParam.checkWay"
                  :showSearch="true"
                  optionFilterProp="label"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="1" label="内检">内检</a-select-option>
                  <a-select-option value="2" label="外检">外检</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="检验类型">
                <a-select
                  v-model.trim="queryParam.checkType"
                  :showSearch="true"
                  optionFilterProp="label"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="" label="">全部</a-select-option>
                  <a-select-option value="1" label="定期检验">定期检验</a-select-option>
                  <a-select-option value="2" label="监督检验">监督检验</a-select-option>
                </a-select>
                <!-- <j-dict-select-tag
                  type="list"
                  v-model="queryParam.checkType"
                  dictCode="checkPlanType"
                  placeholder="请选择"
                /> -->
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!-- <a-button type="primary" @click="visible = true" icon="filter" style="margin-left: 8px">更多</a-button> -->
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
      <a-button @click="handleExecute" type="primary">执行检验</a-button>
      <a-button @click="handleAdjust" type="primary">调整计划执行时间</a-button>
      <a-button type="primary" icon="import" @click="handleExportXls('检验计划表')">导出</a-button>
      <!-- <a-uploads
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-uploads> -->
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
      <a-radio-group @change="searchQuery" v-model="queryParam.overdue" button-style="solid">
        <a-radio-button style="color: #000cff;margin-right:10px" value="1"
          >本月到期({{ this.status1Num }})</a-radio-button
        >
        <a-radio-button style="color: #3bd938;margin-right:10px" value="2"
          >下月到期({{ this.status2Num }})</a-radio-button
        >
        <a-radio-button style="color: #ff00a9" value="3">已逾期({{ this.status3Num }})</a-radio-button>
      </a-radio-group>
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
          <a @click="handleBlockUp(record)">{{ record.isForbid == '1' ? '启用' : '停用' }}</a>
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

    <tab-checkout-plan-modal ref="modalForm" @ok="modalFormOk"></tab-checkout-plan-modal>
    <!-- 调整计划执行时间 -->
    <j-modal title="调整计划执行时间" width="600px" :visible="adjustVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="时间调整为">
                <a-date-picker
                  showTime
                  valueFormat="YYYY-MM-DD HH:mm:ss"
                  style="width: 100%"
                  v-model="adjustData.nextCheckTime"
                />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="adjustOk">确认</a-button>
      </template>
    </j-modal>
  </a-card>
</template>

<script>
import { getAction, putAction, postAction } from '@/api/manage'
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabCheckoutPlanModal from './modules/TabCheckoutPlanModal'

export default {
  name: 'TabCheckoutPlanList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabCheckoutPlanModal
  },
  data() {
    return {
      description: '检验计划表管理页面',
      queryTime: [],
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
          title: '规格型号',
          align: 'center',
          dataIndex: 'deviceModel'
        },
        {
          title: '上次检验时间',
          align: 'center',
          dataIndex: 'lastCheckTime'
        },
        {
          title: '下次检验时间',
          align: 'center',
          dataIndex: 'nextCheckTime'
        },
        {
          title: '检验方式',
          align: 'center',
          dataIndex: 'checkWay'
        },
        {
          title: '检验类型',
          align: 'center',
          dataIndex: 'checkType'
        },
        {
          title: '循环周期',
          align: 'center',
          dataIndex: 'cycleTime'
        },

        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
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
        list: '/checkout/tabCheckoutPlan/list',
        delete: '/checkout/tabCheckoutPlan/delete',
        edit: '/checkout/tabCheckoutPlan/edit',
        deleteBatch: '/checkout/tabCheckoutPlan/deleteBatch',
        exportXlsUrl: '/checkout/tabCheckoutPlan/exportXls',
        queryById: '/checkout/tabCheckoutPlan/queryById?id=', //queryById
        modifyStatus: '/checkout/tabCheckoutPlan/modifyStatus', //启用、停用
        renewTimes: '/checkout/tabCheckoutPlan/renewTimes', //调整执行时间
        importExcelUrl: 'checkout/tabCheckoutPlan/importExcel'
      },
      status1Num: 0,
      status2Num: 0,
      status3Num: 0,
      viewData: {},
      dictOptions: {},
      /* table选中keys*/
      selectedRowKeys: [],
      /* table选中records*/
      selectionRows: [],
      adjustData: {},
      adjustVisible: false,
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
    handleBlockUp(record) {
      // 是否禁用 0：未禁用 1已禁用
      var pm = new URLSearchParams()
      pm.append('id', record.id)
      if (record.isForbid == '1') {
        pm.append('status', '0')
      } else {
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
    },
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
      getAction('/checkout/tabCheckoutPlan/GetCheckPlanStatus').then(res => {
        let data = res.result
        // console.log('保养计划', data)
        // 状态 currentMonthStatus:本月到期 nextMonthStatus:下月到期 overStatus:已逾期
        data.forEach(item => {
          if (item.name == 'currentMonthStatus') {
            this.status1Num = item.num
          }
          if (item.name == 'nextMonthStatus') {
            this.status2Num = item.num
          }
          if (item.name == 'overStatus') {
            this.status3Num = item.num
          }
        })
        console.log('this.status3Num', this.status1Num)
      })
    },
    handleExecute() {
      if (this.selectedRowKeys.length == 1) {
        this.$refs.modalForm.execute(this.selectionRows)
        this.$refs.modalForm.title = '执行检验计划'
        this.$refs.modalForm.disableSubmit = false
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    handleEdit(record) {
      this.handleData(record.id)
      setTimeout(() => {
        this.$refs.modalForm.edit(this.viewData)
        this.$refs.modalForm.title = '编辑'
        this.$refs.modalForm.disableSubmit = false
      }, 500)
    },
    handleDetail(record) {
      this.handleData(record.id)
      setTimeout(() => {
        this.$refs.modalForm.edit(this.viewData)
        this.$refs.modalForm.title = '详情'
        this.$refs.modalForm.disableSubmit = true
      }, 500)
    },
    handleData(id) {
      getAction(this.url.queryById + id).then(res => {
        if (res.success) {
          this.viewData = res.result
          console.log(' this.viewData', this.viewData)
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    handleAdjust() {
      console.log('this.selectedRowKeys', this.selectedRowKeys)
      if (this.selectedRowKeys.length > 1) {
        this.adjustVisible = true
      } else {
        this.$message.warning('请至少选中一条数据！')
      }
    },
    handleCancel() {
      this.adjustData = {}
      this.adjustVisible = false
    },
    adjustOk() {
      var id = this.selectedRowKeys.map(item => item)
      console.log('this.selectedRowKeys', this.selectedRowKeys)
      this.adjustData.id = id
      postAction(this.url.renewTimes, this.adjustData).then(res => {
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
        this.handleCancel()
        this.loadData()
      })
    },
    searchReset() {
      this.queryTime = []
      this.queryParam = {}
      this.queryParam.startTime = ''
      this.queryParam.endTime = ''
      this.loadData(1)
    },
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.startTime = dateString[0]
      this.queryParam.endTime = dateString[1]
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'delFlag', text: '删除标志', dictCode: '' })
      fieldList.push({ type: 'string', value: 'checkWay', text: '检验方式', dictCode: '' })
      fieldList.push({ type: 'string', value: 'checkType', text: '检验类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'cycleTime', text: '循环周期', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'nextCheckTime', text: '下次检验时间' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
