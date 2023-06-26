<template>
  <div class="page-header-index-wide">
    <a-row :gutter="24" style="background: #F0F2F5;height:100%">
      <!-- 工序任务 -->
      <a-col :sm="12" :md="8" :xs="12" :xl="6" style="height:100%">
        <a-card style="height: 86vh; overflow-y: scroll; overflow-x: hidden">
          <div style="position: relative">
            <span style="display: block; font-size:20px; font-weight:600; color: #000c17">工序任务</span>
            <div class="date-buttons">
              <a-tabs :default-active-key="2" @change="changeEvent" size="small" style="padding:0 20px">
                <a-tab-pane :key="0" tab="今天"> </a-tab-pane>
                <a-tab-pane :key="1" tab="昨天"> </a-tab-pane>
                <a-tab-pane :key="2" tab="本月"> </a-tab-pane>
              </a-tabs>
            </div>
            <a-checkbox
              style="margin: 15px 0 35px 15px;font-size:14px"
              v-model="processParam.start"
              @change="processLoad"
              >包含未开始生产任务</a-checkbox
            >
          </div>
          <a-col
            :loading="processLoading"
            v-for="(item, index) in processData"
            :key="index"
            :sm="24"
            :md="24"
            :xl="24"
            :style="{ marginBottom: '24px' }"
          >
            <chart-card :loading="loading" :title="item.procedureName">
              <a-row>
                <a-col :span="12">
                  <div class="proc-left">
                    <span style="display: block; margin-bottom: 10px">
                      计划数
                      <span style="color:#000;font-size:16px; ">
                        {{ item.planAmount ? item.planAmount : 0 }}
                      </span>
                    </span>
                    <span style="display: block; margin-bottom: 10px">
                      良品数
                      <span style="color:#02b980;font-size:16px">
                        {{ item.goodsPros ? item.goodsPros : 0 }}
                      </span>
                    </span>
                    <span style="display: block; margin-bottom: 10px">
                      不良品数
                      <span style="color:#faad14;font-size:16px">
                        {{ item.rejects ? item.rejects : 0 }}
                      </span>
                    </span>
                  </div>
                </a-col>
                <a-col :span="12" style="border-left: 1px solid #f9f7f7;">
                  <div class="proc-left">
                    <p style="color:#000;font-size:36px">{{ item.workOrderCount ? item.workOrderCount : 0 }}</p>
                    <p style="margin-top: -20px">生产任务数</p>
                  </div>
                </a-col>
              </a-row>
              <template slot="footer">
                <div style="width：80%">
                  <a-progress :percent="item.taskProgress" status="active" />
                </div>
              </template>
            </chart-card>
          </a-col>
          <a-col v-if="processData.length == 0">
            <div style="font-size: 16px; color: #000; padding: 50px; text-align: center">
              暂无数据
            </div>
          </a-col>
        </a-card>
      </a-col>
      <a-col :sm="12" :md="16" :xs="12" :xl="18" style="height:100%">
        <a-row :gutter="24" style="margin-bottom: 25px;height:20%">
          <!-- 绩效指标 -->
          <a-col :md="16">
            <a-card>
              <div>
                <span style="display: block; font-size:20px; font-weight:600; color: #000c17">绩效指标</span>
              </div>
              <a-row>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>计划数</p>
                    </div>
                    <div class="item-content">
                      <span style="font-size: 28px; color: rgb(0, 0, 0); margin-top: 20px;">
                        {{ this.performData.planAmount ? this.performData.planAmount : 0 }}
                      </span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>工单数量</p>
                    </div>
                    <div class="item-content">
                      <span style="font-size: 28px; color: rgb(0, 0, 0); margin-top: 20px;">
                        {{ this.performData.workOrderAmount ? this.performData.workOrderAmount : 0 }}
                      </span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>平均工单生产周期</p>
                    </div>
                    <div class="item-content">
                      <span style="font-size: 28px; color: #000000; margin-top: 20px; ">
                        {{
                          this.performData.averageWorkOrderProductionCycle
                            ? this.performData.averageWorkOrderProductionCycle
                            : 0
                        }}天
                      </span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>完成数量</p>
                    </div>
                    <div class="item-content">
                      <span style="font-size: 28px; color: #22AC38; margin-top: 20px;">
                        {{ this.performData.completeAmount ? this.performData.completeAmount : 0 }}
                      </span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>不良品率</p>
                    </div>
                    <div class="item-content">
                      <span style="font-size: 28px; color: #F39800;; margin-top: 20px;">
                        {{ this.performData.rejectsRate ? this.performData.rejectsRate : 0 }}%
                      </span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="item">
                    <div class="item-header">
                      <p>超期未完成工单</p>
                    </div>
                    <div class="item-content">
                      <span
                        style="font-size: 28px;
                      color: #e91e63;
                      margin-top: 20px;
                      "
                      >
                        {{
                          this.performData.overdueUnfinishedWorkOrder ? this.performData.overdueUnfinishedWorkOrder : 0
                        }}
                      </span>
                    </div>
                  </div>
                </a-col>
              </a-row>
            </a-card>
          </a-col>
          <!-- 生产实时播报 -->
          <a-col :md="8">
            <a-card style="height: 398px">
              <div style="height:398px">
                <div>
                  <span style="display: block; font-size:20px; font-weight:600; color: #000c17">生产实时播报</span>
                </div>
                <div style="overflow-y: auto;overflow-x:hidden;height:320px">
                  <div v-for="(item, index) in socketData" :key="index" style="padding-left:20px">
                    <div class="circle"></div>
                    <span>{{ item.date }}</span>
                    <a-tooltip>
                      <template #title> {{ item.userName }}</template>
                      <div class="prod-content">
                        <p style="color:#000;font-size:14px;">{{ item.userName }} {{ item.job }}</p>
                        <p>
                          <span>工单号:</span><span>{{ item.workOrder }}</span>
                        </p>
                        <p>
                          <span>产品信息:</span><span>{{ item.productInformation }}</span>
                        </p>
                        <p>
                          <span>{{ item.goods }} 良品 / {{ item.bad }} 不良品</span>
                        </p>
                      </div>
                    </a-tooltip>
                  </div>
                  <div
                    style="font-size: 16px; color: #000; padding: 50px; text-align: center"
                    v-if="socketData.length == 0"
                  >
                    暂无数据
                  </div>
                </div>
              </div>
            </a-card>
          </a-col>
        </a-row>
        <!-- 工单进度报表 -->
        <a-card
          :loading="loading"
          :class="{ 'anty-list-cust': true }"
          :bordered="false"
          style="height:43vh;padding-bottom:30px"
        >
          <a-row>
            <a-col :xl="16" :lg="16" :md="16" :sm="16" :xs="16" :style="{ paddingRight: '10px' }">
              <span style="display: block; font-size:20px; font-weight:600; color: #000c17">工单进度报表</span>
            </a-col>
            <a-col :xl="8" :lg="8" :md="8" :sm="8" :xs="8" :style="{ paddingRight: '10px' }">
              <a-radio-group v-model="workStatus" @change="changeRegisterType">
                <a-radio-button value="">全部</a-radio-button>
                <a-radio-button value="0">未开始</a-radio-button>
                <a-radio-button value="10">执行中</a-radio-button>
                <a-radio-button value="100">已结束</a-radio-button>
              </a-radio-group>
            </a-col>
          </a-row>
          <a-table
            ref="table"
            size="middle"
            :scroll="{ x: true, y: 250 }"
            bordered
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :loading="tableLoading"
            :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
            class="j-table-force-nowrap"
            style="margin-top: 20px"
            @change="handleTableChange"
          >
            <template slot="planAmount" slot-scope="text, record">
              <a-tooltip>
                <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left">
                  <a-progress type="circle" :width="50" :percent="item.percentage" />
                  <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
                  <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
                </div>
              </a-tooltip>
            </template>
            <template slot="flowRate" slot-scope="text, record">
              <a-progress
                :strokeColor="getPercentColor(record.flowRate)"
                :format="getPercentFormat"
                :percent="getFlowRateNumber(record.flowRate)"
                style="width:80px"
              />
            </template>

            <template slot="action" slot-scope="text, record">
              <a @click="workHandle(record.id, 'start')" v-has="'SysWorkOrderList:start'">开始</a>
              <a-divider type="vertical" v-has="'SysWorkOrderList:start'" />
              <a @click="workHandle(record.id, 'withdraw')" v-has="'SysWorkOrderList:recall'">撤回</a>
              <br />
              <a @click="workHandle(record.id, 'finish')" v-has="'SysWorkOrderList:end'">结束</a>
              <a-divider type="vertical" v-has="'SysWorkOrderList:end'" />
              <a-popconfirm
                v-has="'SysWorkOrderList:delete'"
                title="确认删除工单？操作将同时删除关联的任务和报工，删除后不可恢复"
                @confirm="() => handleDelete(record.id)"
              >
                <a>删除</a>
              </a-popconfirm>
            </template>
          </a-table>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>

<script>
import ChartCard from '@/components/ChartCard'
import ACol from 'ant-design-vue/es/grid/Col'
import ATooltip from 'ant-design-vue/es/tooltip/Tooltip'
import MiniArea from '@/components/chart/MiniArea'
import MiniBar from '@/components/chart/MiniBar'
import MiniProgress from '@/components/chart/MiniProgress'
import Bar from '@/components/chart/Bar'
import HeadInfo from '@/components/tools/HeadInfo.vue'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, httpAction } from '@/api/manage'
import store from '@/store/'
import Trend from '@/components/Trend'
import { mixin, mixinDevice } from '@/utils/mixin.js'
export default {
  name: 'IndexCustom',
  mixins: [JeecgListMixin, mixin, mixinDevice],
  components: {
    ATooltip,
    ACol,
    ChartCard,
    MiniArea,
    MiniBar,
    MiniProgress,
    Bar,
    Trend,
    HeadInfo
  },
  data() {
    return {
      dataSource: [],
      processData: [],
      performData: [],
      processParam: {
        date: '2',
        start: false
      },
      url: {
        list: '/longxianda/sysWorkOrder/list', //工单进度报表
        delete: '/longxianda/sysWorkReporting/deleteWorkOrder',
        performList: '/longxianda/sysWorkOrderDetail/homePerformanceIndicators', //绩效指标
        processList: '/longxianda/sysWorkOrderDetail/homeWorkProcedure' //工序
      },
      queryParam: { superQueryParams: '', superQueryMatchType: 'and' },
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
          title: '工单编号',
          align: 'center',
          dataIndex: 'code',
          width: 140,
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品编码',
          align: 'center',
          width: 140,
          dataIndex: 'productCode',
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品名称',
          align: 'center',
          width: 140,
          dataIndex: 'productName',
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品规格',
          align: 'center',
          width: 140,
          dataIndex: 'productUnit',
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划数',
          align: 'center',
          width: 140,
          dataIndex: 'planAmount',
          ellipsis: true,
          resizable: true
        },
        {
          title: '完工数',
          align: 'center',
          width: 140,
          dataIndex: 'offNum',
          ellipsis: true,
          resizable: true
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'status',
          width: 120,
          customRender: function(t, r, index) {
            if (r.status == 0) {
              return '未开始'
            } else if (r.status == 10) {
              return '执行中'
            } else {
              return '已结束'
            }
          }
        },
        {
          title: '计划开始时间',
          align: 'center',
          width: 160,
          dataIndex: 'planStartTime',
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划结束时间',
          align: 'center',
          width: 160,
          dataIndex: 'planEndTime',
          ellipsis: true,
          resizable: true
        },
        {
          title: '生产进度',
          align: 'center',
          width: 500,
          scopedSlots: { customRender: 'planAmount' },
          ellipsis: true,
          resizable: true
        },
        {
          title: '备注',
          align: 'center',
          width: 150,
          dataIndex: 'remarks',
          ellipsis: true,
          resizable: true
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 150,
          scopedSlots: { customRender: 'action' }
        }
      ],
      productionSchedule: [
        {
          title: '开料',
          percentage: 25
        },
        {
          title: '打包',
          percentage: 45
        },
        {
          title: '清洁',
          percentage: 90
        },
        {
          title: '打孔',
          percentage: 10
        },
        {
          title: '试装',
          percentage: 100
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
      socketData: [],
      workStatus: '',
      indexBottomTab: '1',
      loading: false,
      tableLoading: false,
      processLoading: false,
      performLoading: false,
      center: null
    }
  },
  created() {
    this.initWebSocket()
    console.log('color', this.primaryColor)
    this.processLoad() //工序任务
    this.performLoad() //绩效指标
    this.socketFirst() //生产实时播报
  },
  destroyed() {
    // 离开页面生命周期函数
    this.websocketOnclose()
  },
  methods: {
    workHandle(id, type) {
      var that = this
      that.confirmLoading = true
      var httpurl = ''
      if (type == 'start') {
        httpurl += '/longxianda/sysWorkOrder/workStart?id=' + id
      } else if (type == 'withdraw') {
        httpurl += '/longxianda/sysWorkOrder/workWithdraw?id=' + id
      } else {
        httpurl += '/longxianda/sysWorkOrder/workFinish?id=' + id
      }
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            that.$message.warning(res.message)
            this.loadData()
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
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
      this.tableLoading = true
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
          this.tableLoading = false
        })
    },
    // socket
    initWebSocket: function() {
      // WebSocket与普通的请求所用协议有所不同，ws等同于http，wss等同于https
      var userId = store.getters.userInfo.id
      // ws://192.168.56.61:8080/jeecg-boot/websocket/admin
      var url =
        window._CONFIG['domianURL'].replace('https://', 'wss://').replace('http://', 'ws://') + '/websocket/' + userId
      console.log(url)
      this.websock = new WebSocket(url)
      this.websock.onopen = this.websocketOnopen
      this.websock.onerror = this.websocketOnerror
      this.websock.onmessage = this.websocketOnmessage
      this.websock.onclose = this.websocketOnclose
      // this.socketFirst()
    },
    websocketOnopen: function() {
      console.log('WebSocket连接成功')
      //心跳检测重置
      //this.heartCheck.reset().start();
    },
    websocketOnerror: function(e) {
      console.log('WebSocket连接发生错误')
      this.reconnect()
    },
    websocketOnmessage: function(e) {
      console.log('-----接收消息-------', e.data)
      // var data = eval('(' + e.data + ')') //解析对象
      var data = eval(e.data) //解析对象
      console.log('data', data)
      data.forEach(item => {
        this.socketData.unshift(item)
      })
      console.log('this.接收消息socketData', this.socketData)

      if (data.cmd == 'topic') {
        //系统通知
        // this.loadData()
      } else if (data.cmd == 'user') {
        //用户消息
        // this.loadData()
      }
      //心跳检测重置
      //this.heartCheck.reset().start();
    },
    websocketOnclose: function(e) {
      console.log('connection closed (' + e + ')')
      if (e) {
        console.log('connection closed (' + e.code + ')')
      }
      this.reconnect()
      console.log('销毁socket-------')
    },
    reconnect() {
      var that = this
      if (that.lockReconnect) return
      that.lockReconnect = true
      //没连接上会一直重连，设置延迟避免请求过多
      setTimeout(function() {
        console.info('尝试重连...')
        that.initWebSocket()
        that.lockReconnect = false
      }, 5000)
    },
    // end
    socketFirst() {
      getAction('/sys/socketTest/sendUser?userId=' + store.getters.userInfo.id).then(res => {
        console.log('socketFirst', res)
        // let data = eval(res.result)
      })
    },
    changeEvent(activeKey) {
      this.processParam.date = activeKey
      this.processLoad() //工序任务
      this.performLoad() //绩效指标
    },
    processLoad() {
      this.processLoading = true
      console.log('this.processParam', this.processParam)
      getAction(this.url.processList, this.processParam)
        .then(res => {
          if (res.success) {
            this.processData = res.result || []
            if (res.result != '今日没有订单' && res.result != '昨日没有订单' && res.result != '本月没有订单') {
              this.processData = res.result || []
            } else {
              this.processData = []
            }
            // console.log('res.result.length', res.result.length)
            // console.log('res.result', res.result)
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.processLoading = false
        })
    },
    performLoad() {
      this.performLoading = true
      getAction(this.url.performList, { date: this.processParam.date })
        .then(res => {
          if (res.success) {
            this.performData = res.result || []
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.performLoading = false
        })
    },
    getPercentFormat(value) {
      if (value == 100) {
        return '超时'
      } else {
        return value + '%'
      }
    },
    getFlowRateNumber(value) {
      return Number(value)
    },
    getPercentColor(value) {
      let p = Number(value)
      if (p >= 90 && p < 100) {
        return 'rgb(244, 240, 89)'
      } else if (p >= 100) {
        return 'red'
      } else {
        return 'rgb(16, 142, 233)'
      }
    },
    changeRegisterType(e) {
      this.queryParam.superQueryParams = e.target.value
      this.workStatus = e.target.value
      let sqp = {}
      if (this.queryParam) {
        let params = [
          { rule: 'eq', type: 'string', dictCode: '', val: this.queryParam.superQueryParams, field: 'status' }
        ]
        params = JSON.stringify(params)
        sqp['superQueryParams'] = encodeURI(params)
        sqp['superQueryMatchType'] = this.queryParam.superQueryMatchType
      }
      this.queryParam = sqp
      this.loadData()
    }
  }
}
</script>

<style lang="less" scoped>
::v-deep .ant-progress-outer {
  width: 90%;
}

.proc-left {
  height: 90px;
  font-size: 14px;
  text-align: center;
}
.prod-title {
  color: #000000;
}
.circle {
  display: inline-block;
  border: 1px solid rgb(184, 116, 116);
  background-color: transparent;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 10px;
}
.prod-content {
  width: 90%;
  margin: 10px;
  height: 130px;
  margin-left: 20px;
  padding: 10px 0 20px 10px;
  font-size: 11px;
  background-color: #fafafa;
  overflow: hidden;
}
.item-group {
  height: 50px;
  padding: 10px 0 0 20px;
  font-size: 16px;
  color: #000;
}
.item {
  text-align: center;
  margin: 15px 15px 0 0;
}
// .item-header {
//   width: 100%;
//   height: 24px;
//   font-size: 14px;
//   line-height: 24px;
//   color: #fff;
//   border-radius: 5px 5px 0 0;
// }
.item-content {
  height: 120px;
  line-height: 110px;
  text-align: center;
  background-color: #fafafa;
}
.circle-cust {
  position: relative;
  top: 28px;
  left: -100%;
}
.extra-wrapper {
  line-height: 55px;
  padding-right: 24px;

  .extra-item {
    display: inline-block;
    margin-right: 24px;

    a {
      margin-left: 24px;
    }
  }
}
.date-buttons {
  position: absolute;
  top: -6px;
  right: -44px;
  width: 80%;
}
/* 首页访问量统计 */
.head-info {
  position: relative;
  text-align: left;
  padding: 0 32px 0 0;
  min-width: 125px;

  &.center {
    text-align: center;
    padding: 0 32px;
  }

  span {
    color: rgba(0, 0, 0, 0.45);
    display: inline-block;
    font-size: 0.95rem;
    line-height: 42px;
    margin-bottom: 4px;
  }
  p {
    line-height: 42px;
    margin: 0;
    a {
      font-weight: 600;
      font-size: 1rem;
    }
  }
}
</style>
