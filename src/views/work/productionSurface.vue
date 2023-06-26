<template>
  <div class="banner">
    <div style="height:70%">
      <div class="headers">
        <div class="headers-title"></div>
        <span class="title-out" @click="ScreenVis('on')">全屏</span>
        <span class="title-quanpin-icon" @click="ScreenVis('on')"></span>
      </div>
      <div class="banner-content">
        <a-col :span="7" class="content">
          <div class="left-menu-title">
            <span style="margin-left:60px">工单产出分析</span>
            <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;"
              >近10天</span
            >
            <div id="mains"></div>
            <div id="main-2"></div>
          </div>
        </a-col>
        <a-col :span="10">
          <div class="center-menu-content">
            <div class="top">
              <a-col :span="12">
                <div class="content-middle">
                  <div class="number-icon"></div>
                  <div style="float:right;margin-top: -20px;">
                    <span>在制工单数 </span>
                    <span class="number">{{ model.workInProcess }}</span>
                    <span class="number-top">个</span>
                  </div>
                </div>
              </a-col>
              <a-col :span="12">
                <div class="content-middle">
                  <div class="renwu-icon"></div>
                  <div style="float:right;margin-top: -20px;">
                    <span>在制任务数 </span>
                    <span class="number">{{ model.taskInProcess }}</span>
                    <span class="number-top">个</span>
                  </div>
                </div>
              </a-col>
            </div>
            <div class="center">
              <div class="center-left">
                <div class="icon-l"></div>
                <div class="data">
                  <div class="content-data">
                    <div class="middle">
                      <span class="icon-span"></span>
                      <span>在制工单计划数</span>
                      <br />
                      <span class="data-number">
                        {{ model.workPlanInProcess ? model.workPlanInProcess : 0 }}
                        <span class="data-succuss">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span class="icon-span"></span>
                      <span>在制工单延期数</span>
                      <br />
                      <span class="data-number-error">
                        {{ model.delayWorkInProcess ? model.delayWorkInProcess : 0 }}
                        <span class="data-error">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span class="icon-span"></span>
                      <span>今日完成工单数</span>
                      <br />
                      <span class="data-number">
                        {{ model.finishWork ? model.finishWork : 0 }}
                        <span class="data-succuss">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span class="icon-span"></span>
                      <span>今日工单产出数</span>
                      <br />
                      <span class="data-number">
                        {{ model.workOutput ? model.workOutput : 0 }}
                        <span class="data-succuss">个</span>
                      </span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="center-right">
                <div class="icon-l"></div>
                <div class="data">
                  <div class="content-data">
                    <div class="middle">
                      <span>在制任务计划数</span>
                      <span class="icon-span"></span>
                      <br />
                      <span class="data-number">
                        {{ model.taskPlanInProcess ? model.taskPlanInProcess : 0 }}
                        <span class="data-succuss">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span>在制任务延期数</span>
                      <span class="icon-span"></span>
                      <br />
                      <span class="data-number-error">
                        {{ model.delayWorkInProcess ? model.delayWorkInProcess : 0 }}
                        <span class="data-error">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span>今日完成任务数</span>
                      <span class="icon-span"></span>
                      <br />
                      <span class="data-number">
                        {{ model.finishTask ? model.finishTask : 0 }}
                        <span class="data-succuss">个</span>
                      </span>
                    </div>
                    <div class="middle">
                      <span>今日不良品率</span>
                      <span class="icon-span"></span>
                      <br />
                      <span class="data-number-error">
                        {{ model.defectRate ? model.defectRate : 0 }}
                        <span class="data-error">%</span>
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="footer">
              <ul class="scroll-wrapper-ul">
                <li class="scroll-wrapper-li" v-for="(item, index) in model.workRepor" :key="index">
                  <div class="prefix_APscC"></div>
                  <span>{{ item.createTime }}</span>
                  <span>{{ item.operatorName }}|{{ item.outputNumber }}</span>
                  <span> {{ item.productName }}|{{ item.processName }} </span>
                  <span> 良品: {{ item.fineNumber }}</span>
                  <span>不良品: {{ item.defectsNumber }}</span>
                </li>
              </ul>
            </div>
          </div>
        </a-col>
        <a-col :span="7" class="content">
          <div class="left-menu-title">
            <span style="margin-left:60px">不良品分析</span>
            <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;"
              >近10天</span
            >
            <div id="main-3"></div>
            <div id="main-4"></div>
          </div>
        </a-col>
      </div>
    </div>
    <div class="content-table">
      <a-col :span="17" class="content">
        <div class="work-order-list-1">
          <span style="margin-left:6%;display: inline-block;margin-bottom:20px">工单概览</span>
          <div style="height:20vh;overflow-y:auto">
            <a-table
              rowKey="id"
              :columns="workOrderColumns"
              :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
              :data-source="model.workOrderList"
              :bordered="false"
              :pagination="false"
            >
              <template slot="planAmountSlot" slot-scope="text, record">
                <a-tooltip>
                  <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left">
                    <a-progress type="circle" :width="30" :percent="item.percentage" />

                    <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
                    <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
                  </div>
                </a-tooltip>
              </template>
            </a-table>
          </div>
        </div>
      </a-col>
      <a-col :span="7" class="content">
        <div class="work-order-list">
          <span style="margin-left:10%;display: inline-block;margin-bottom:20px">任务执行进度</span>
          <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;">今天</span>
          <div style="height:20vh;overflow-y:auto">
            <a-table
              rowKey="id"
              :columns="taskExecuteColumns"
              :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
              :data-source="model.taskExecute"
              :bordered="false"
              :pagination="false"
            >
            </a-table>
          </div>
        </div>
      </a-col>
    </div>
    <j-modal
      title="弹窗"
      width="600"
      :visible="visible"
      :fullscreen="fullscreen"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div style="height:70%">
        <div class="headers">
          <div class="headers-title"></div>
          <span class="title-out" @click="ScreenVis('out')">退出全屏</span>
          <span class="title-out-icon" @click="ScreenVis('out')"></span>
        </div>
        <div class="banner-content">
          <a-col :span="7" class="content">
            <div class="left-menu-title">
              <span style="margin-left:60px">工单产出分析</span>
              <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;"
                >近10天</span
              >
              <div id="mainA"></div>
              <div id="mainB"></div>
            </div>
          </a-col>
          <a-col :span="10">
            <div class="center-menu-content">
              <div class="top">
                <a-col :span="12">
                  <div class="content-middle">
                    <div class="number-icon"></div>
                    <div style="float:right;margin-top: -20px;">
                      <span>在制工单数 </span>
                      <span class="number">{{ model.workInProcess }}</span>
                      <span class="number-top">个</span>
                    </div>
                  </div>
                </a-col>
                <a-col :span="12">
                  <div class="content-middle">
                    <div class="renwu-icon"></div>
                    <div style="float:right;margin-top: -20px;">
                      <span>在制任务数 </span>
                      <span class="number">{{ model.taskInProcess }}</span>
                      <span class="number-top">个</span>
                    </div>
                  </div>
                </a-col>
              </div>
              <div class="center">
                <div class="center-left">
                  <div class="icon-l"></div>
                  <div class="data">
                    <div class="content-data">
                      <div class="middle">
                        <span class="icon-span"></span>
                        <span>在制工单计划数</span>
                        <br />
                        <span class="data-number">
                          {{ model.workPlanInProcess ? model.workPlanInProcess : 0 }}
                          <span class="data-succuss">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span class="icon-span"></span>
                        <span>在制工单延期数</span>
                        <br />
                        <span class="data-number-error">
                          {{ model.delayWorkInProcess ? model.delayWorkInProcess : 0 }}
                          <span class="data-error">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span class="icon-span"></span>
                        <span>今日完成工单数</span>
                        <br />
                        <span class="data-number">
                          {{ model.finishWork ? model.finishWork : 0 }}
                          <span class="data-succuss">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span class="icon-span"></span>
                        <span>今日工单产出数</span>
                        <br />
                        <span class="data-number">
                          {{ model.workOutput ? model.workOutput : 0 }}
                          <span class="data-succuss">个</span>
                        </span>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="center-right">
                  <div class="icon-l"></div>
                  <div class="data">
                    <div class="content-data">
                      <div class="middle">
                        <span>在制任务计划数</span>
                        <span class="icon-span"></span>
                        <br />
                        <span class="data-number">
                          {{ model.taskPlanInProcess ? model.taskPlanInProcess : 0 }}
                          <span class="data-succuss">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span>在制任务延期数</span>
                        <span class="icon-span"></span>
                        <br />
                        <span class="data-number-error">
                          {{ model.delayWorkInProcess ? model.delayWorkInProcess : 0 }}
                          <span class="data-error">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span>今日完成任务数</span>
                        <span class="icon-span"></span>
                        <br />
                        <span class="data-number">
                          {{ model.finishTask ? model.finishTask : 0 }}
                          <span class="data-succuss">个</span>
                        </span>
                      </div>
                      <div class="middle">
                        <span>今日不良品率</span>
                        <span class="icon-span"></span>
                        <br />
                        <span class="data-number-error">
                          {{ model.defectRate ? model.defectRate : 0 }}
                          <span class="data-error">%</span>
                        </span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="footer">
                <ul class="scroll-wrapper-ul">
                  <li class="scroll-wrapper-li" v-for="(item, index) in model.workRepor" :key="index">
                    <div class="prefix_APscC"></div>
                    <span>{{ item.createTime }}</span>
                    <span>{{ item.operatorName }}|{{ item.outputNumber }}</span>
                    <span> {{ item.productName }}|{{ item.processName }} </span>
                    <span> 良品: {{ item.fineNumber }}</span>
                    <span>不良品: {{ item.defectsNumber }}</span>
                  </li>
                </ul>
              </div>
            </div>
          </a-col>
          <a-col :span="7" class="content">
            <div class="left-menu-title">
              <span style="margin-left:60px">不良品分析</span>
              <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;"
                >近10天</span
              >
              <div id="mainC"></div>
              <div id="mainD"></div>
            </div>
          </a-col>
        </div>
      </div>
      <div class="content-table">
        <a-col :span="17" class="content">
          <div class="work-order-list-1">
            <span style="margin-left:6%;display: inline-block;margin-bottom:20px">工单概览</span>
            <div style="height:20vh;overflow-y:auto">
              <a-table
                rowKey="id"
                :columns="workOrderColumns"
                :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
                :data-source="model.workOrderList"
                :bordered="false"
                :pagination="false"
              >
                <template slot="planAmountSlot" slot-scope="text, record">
                  <a-tooltip>
                    <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left">
                      <a-progress type="circle" :width="30" :percent="item.percentage" />

                      <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
                      <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
                    </div>
                  </a-tooltip>
                </template>
              </a-table>
            </div>
          </div>
        </a-col>
        <a-col :span="7" class="content">
          <div class="work-order-list">
            <span style="margin-left:10%;display: inline-block;margin-bottom:20px">任务执行进度</span>
            <span style="float:right;margin-right: 10%;display: inline-block;font-size:14px;color:#CAD4E6FF;"
              >今天</span
            >
            <div style="height:20vh;overflow-y:auto">
              <a-table
                rowKey="id"
                :columns="taskExecuteColumns"
                :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
                :data-source="model.taskExecute"
                :bordered="false"
                :pagination="false"
              >
              </a-table>
            </div>
          </div>
        </a-col>
      </div>
    </j-modal>
  </div>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { getAction } from '@/api/manage'
import screenfull from 'screenfull'

export default {
  name: 'productionSurface',
  components: {},
  data() {
    return {
      visible: true,
      fullscreen: true,
      // tableHeight: document.documentElement.clientHeight - 260 + 'px',
      isFull: '', //全屏检查
      description: '驾驶舱',
      // 表头
      visitFields: ['ip', 'visit'],
      url: {
        list: '/longxianda/sysCode/list',
        delete: '/longxianda/sysCode/delete',
        deleteBatch: '/longxianda/sysCode/deleteBatch',
        exportXlsUrl: '/longxianda/sysCode/exportXls',
        importExcelUrl: 'longxianda/sysCode/importExcel',
        queryProductPanel: '/longxianda/report/queryProductPanel'
      },
      fullScreen: false, // 是否全屏
      workOrderColumns: [
        {
          title: '工单编号',
          align: 'center',
          dataIndex: 'code'
        },
        {
          title: '工单状态',
          align: 'center',
          dataIndex: 'status',
          ellipsis: true,
          resizable: true,
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
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '产品编号',
          align: 'center',
          dataIndex: 'productUnit'
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount'
        },
        {
          title: '实际数',
          align: 'center',
          dataIndex: 'offNum'
        },
        {
          title: '生产进度',
          align: 'center',
          width: 400,
          scopedSlots: { customRender: 'planAmountSlot' },
          ellipsis: true,
          resizable: true
        },
        {
          title: '剩余数',
          align: 'center',
          // dataIndex: 'offNum',
          customRender: function(t, r, index) {
            return r.planAmount - r.offNum
          },
          ellipsis: true,
          resizable: true
        }
      ],
      taskExecuteColumns: [
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'procedureName'
          // width: 120,
          // ellipsis: true,
          // resizable: true
        },
        {
          title: '生产任务数',
          align: 'center',
          dataIndex: 'taskCount'
          // width: 120,
          // ellipsis: true,
          // resizable: true
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount'
        },
        {
          title: '良品数',
          align: 'center',
          dataIndex: 'fineAmount'
          // width: 120,
          // ellipsis: true,
          // resizable: true
        },
        {
          title: '不良品数',
          align: 'center',
          dataIndex: 'defectsAmount'
          // width: 120,
          // ellipsis: true,
          // resizable: true
        },
        {
          title: '未完成数',
          align: 'center',
          dataIndex: 'unfinishCount'
          // width: 120,
          // ellipsis: true,
          // resizable: true
        }
      ],
      workOrderData: [],
      model: {
        // workOrderList: []
      },
      model: {}
    }
  },
  created() {
    this.queryProductPanel()
  },
  mounted() {
    setTimeout(() => {
      this.getEcharts()
      this.getMainTwo()
      this.getMainTree()
      this.getMainFour()
    }, 500)
  },
  methods: {
    handleCancel() {
      this.visible = false
    },
    ScreenVis(type) {
      if (type == 'on') {
        this.visible = true
      } else {
        this.visible = false
      }
    },
    queryProductPanel() {
      const that = this
      getAction(this.url.queryProductPanel, {}, 'get')
        .then(res => {
          //  that.recordList = res;
          console.log(res)
          that.model = res.result
        })
        .finally(() => {})
    },
    getMainTwo() {
      var that = this
      var myChart = this.$echarts.init(document.getElementById('main-2'))
      var myChartB = this.$echarts.init(document.getElementById('mainB'))
      getAction(this.url.queryProductPanel, {}, 'get').then(res => {
        //  that.recordList = res;
        console.log(res)
        that.model = res.result
        if (that.model.performanceTop.datax.length > 0) {
          var option = {
            title: {
              text: '员工绩效top5',
              textStyle: {
                fontSize: 14,
                color: '#fff'
              }
            },
            textStyle: {
              color: '#fff'
            },
            tooltip: {
              trigger: 'axis',
              axisPointer: {
                type: 'shadow'
              }
            },
            legend: {},
            grid: {
              left: '3%',
              right: '4%',
              bottom: '15%',
              containLabel: true
            },
            xAxis: {
              type: 'value',
              splitLine: {
                show: false,
                lineStyle: {
                  type: 'dashed'
                }
              },
              boundaryGap: [0, 0.01]
            },
            yAxis: {
              type: 'category',
              data: that.model.performanceTop.datay
            },
            series: [
              {
                type: 'bar',
                barWidth: '30%',
                stack: 'total',
                data: that.model.performanceTop.datax,
                markPoint: {
                  symbol: 'image:/img/loop.png', //url替换成你的图片地址
                  symbolSize: 10, //图片大小
                  symbolOffset: [0, -20], //图片相对于柱形图x,y轴偏移量，注意要往上移y要是负的
                  data: [
                    {
                      coord: [0, 8000] //需要展示图片的柱型索引，y轴的value
                    }
                  ],
                  label: {
                    show: false //顶部默认会展示y轴的值，可以设置false,不展示
                  }
                },
                itemStyle: {
                  // 柱状图的背景色
                  normal: {
                    // 每个柱子的颜色即为colorList数组里的每一项,如果柱子数目多于colorList的长度，则柱子颜色循环使用该数组
                    color: function(params) {
                      return {
                        colorStops: [
                          {
                            offset: 0.3, // 颜色的开始位置
                            color: '#2e7dffAD' // 0% 处的颜色
                          },
                          {
                            offset: 0.6, // 颜色的结束位置
                            color: '#2edfffCC' // 100% 处的颜色
                          }
                        ]
                      }
                    }
                  }
                }
              }
            ]
          }
          myChart.setOption(option)
          myChartB.setOption(option)
        }
      })
    },
    getEcharts() {
      var that = this
      var myChart = this.$echarts.init(document.getElementById('mains'))
      var myChartA = this.$echarts.init(document.getElementById('mainA'))
      getAction(this.url.queryProductPanel, {}, 'get').then(res => {
        //  that.recordList = res;
        console.log(res)
        that.model = res.result
        if (that.model.workOrderProcess.datax.length > 0) {
          var option = {
            title: {
              text: '工单产出趋势图',
              textStyle: {
                fontSize: 14,
                color: '#fff'
              }
            },
            textStyle: {
              color: '#fff'
            },
            tooltip: {
              trigger: 'axis',
              axisPointer: {
                type: 'cross',
                crossStyle: {
                  color: '#999'
                }
              }
            },
            legend: {
              data: [
                {
                  name: '完成工单',
                  textStyle: {
                    color: '#fff'
                  }
                },
                {
                  name: '工单产出',
                  textStyle: {
                    color: '#fff'
                  }
                }
              ],
              left: '70%',
              orient: 'vertical'
            },
            xAxis: [
              {
                type: 'category',
                data: that.model.workOrderProcess.datax,
                axisPointer: {
                  type: 'shadow'
                }
              }
            ],
            yAxis: [
              //当有两个y轴时，设置两个{}{}来承载y轴的值
              {
                type: 'value',
                min: 0,
                axisLabel: {
                  color: '#ddd',
                  textStyle: {
                    fontSize: 14,
                    fontWeight: 'normal'
                  }
                },
                splitLine: {
                  color: '#ddd',
                  show: false,
                  lineStyle: {
                    type: 'dashed'
                  }
                }
              },
              {
                type: 'value',
                axisLabel: {
                  textStyle: {
                    fontSize: 14,
                    fontWeight: 'normal'
                  }
                },
                splitLine: {
                  show: false,
                  lineStyle: {
                    type: 'dashed'
                  }
                },
                inverse: true
              }
            ],
            series: [
              {
                name: '完成工单',
                type: 'bar',
                barWidth: '12',
                color: '#3AA5FFFF',
                data: that.model.workOrderProcess.datay1
              },
              {
                name: '工单产出',
                type: 'line',
                yAxisIndex: 1,
                color: '#FDFF7DFF',
                data: that.model.workOrderProcess.datay2,
                label: {
                  normal: {
                    show: false
                  }
                },
                lineStyle: {
                  normal: {
                    width: 1
                  }
                }
              }
            ]
          }
          myChart.setOption(option)
          myChartA.setOption(option)
        }
      })
    },
    getMainTree() {
      var that = this
      var myChart = this.$echarts.init(document.getElementById('main-3'))
      var myChartC = this.$echarts.init(document.getElementById('mainC'))
      getAction(this.url.queryProductPanel, {}, 'get').then(res => {
        //  that.recordList = res;
        console.log(res)
        that.model = res.result
        if (that.model.defectAnalyse.datax.length > 0) {
          var option = {
            title: {
              text: '不良品趋势图',
              textStyle: {
                fontSize: 14,
                color: '#fff'
              }
            },
            textStyle: {
              color: '#fff'
            },
            tooltip: {
              trigger: 'axis',
              axisPointer: {
                type: 'cross',
                crossStyle: {
                  color: '#999'
                }
              }
            },

            legend: {
              data: [
                {
                  name: '不良品数',
                  textStyle: {
                    color: '#fff'
                  }
                },
                {
                  name: '不良品率',
                  textStyle: {
                    color: '#fff'
                  }
                }
              ],
              left: '70%',
              orient: 'vertical'
            },
            xAxis: [
              {
                type: 'category',
                data: that.model.defectAnalyse.datax,
                axisPointer: {
                  type: 'shadow'
                }
              }
            ],
            yAxis: [
              {
                type: 'value',
                // min: 0,
                // interval: 50,
                // axisLabel: {
                //   formatter: '{value}'
                // }
                min: 0,
                axisLabel: {
                  color: '#ddd',
                  textStyle: {
                    fontSize: 14,
                    fontWeight: 'normal'
                  }
                },
                splitLine: {
                  color: '#ddd',
                  show: false,
                  lineStyle: {
                    type: 'dashed'
                  }
                }
              },
              {
                // type: 'value',
                // min: 0,
                // interval: 5,
                // axisLabel: {
                //   formatter: '{value}'
                // }
                axisLabel: {
                  textStyle: {
                    fontSize: 14,
                    fontWeight: 'normal'
                  }
                },
                splitLine: {
                  show: false,
                  lineStyle: {
                    type: 'dashed'
                  }
                },
                inverse: true
              }
            ],
            series: [
              {
                name: '不良品数',
                type: 'bar',
                color: '#3AA5FFFF',
                tooltip: {
                  valueFormatter: function(value) {
                    return value + ''
                  }
                },
                data: that.model.defectAnalyse.datay1
              },
              {
                name: '不良品率',
                type: 'line',
                yAxisIndex: 1,
                color: '#FDFF7DFF',
                tooltip: {
                  valueFormatter: function(value) {
                    return value + '%'
                  }
                },
                data: that.model.defectAnalyse.datay2
              }
            ]
          }
          myChart.setOption(option)
          myChartC.setOption(option)
        }
      })
    },

    // quanpin() {
    //   if (!screenfull.isFullscreen) {
    //     screenfull.toggle()
    //     this.fullScreen = true
    //   } else {
    //     screenfull.toggle()
    //     this.fullScreen = false
    //   }
    // },
    getMainFour() {
      var myChart = this.$echarts.init(document.getElementById('main-4'))
      var myChartD = this.$echarts.init(document.getElementById('mainD'))
      var that = this
      getAction(this.url.queryProductPanel)
        .then(res => {
          console.log('main-4', res)
          that.model = res.result
          console.log('that.model.defectSpread.', that.model.defectSpread)
          if (that.model.defectSpread.length > 0) {
            var option = {
              color: ['#2466FAFF', '#22D8FFFF', '#A7EFFFFF', '#EFEEA6FF', '#FF9E69FF', '#6E8AACFF'],
              title: {
                text: '不良品项分布图',
                left: 'left',
                textStyle: {
                  fontSize: 14,
                  color: '#fff'
                }
              },
              tooltip: {
                trigger: 'item',
                formatter: '{b}: {c} ({d}%)'
              },
              // legend: {
              //   orient: 'vertical',
              //   left: 'right',
              //   textStyle: {
              //     color: '#fff' // 图例文字颜色
              //   }
              // },
              series: [
                {
                  name: that.model.defectSpread.name,
                  type: 'pie',
                  radius: ['30%', '50%'],
                  itemStyle: {
                    normal: {
                      label: {
                        show: true,
                        textStyle: { color: '#fff' }
                      }
                    }
                  },
                  data: that.model.defectSpread
                }
              ]
            }
            myChart.setOption(option)
            myChartD.setOption(option)
          }
        })
        .finally(() => {})
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
::v-deep {
  .ant-modal-body {
    padding: 0;
    height: 100% !important;
  }
  .ant-modal-content {
    background-image: url(img/KB_BG.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
  }
  .ant-modal-content > .ant-modal-header {
    display: none;
  }
  .ant-modal-content > .ant-modal-footer {
    display: none;
  }
  .ant-modal-close {
    display: none;
  }
}
// 固定表头，高度自适应
::v-deep {
  .ant-table-placeholder {
    background: #396ca747;
    color: #fff;
  }
  .ant-empty-normal {
    color: #fff;
  }
}
::v-deep .ant-table-thead > tr > th,
.ant-table-tbody > tr > td {
  padding: 8px 8px;
  overflow-wrap: break-word;
  color: #f5f6ff;
  background: rgba(13, 45, 94, 0.4);
  border: none;
}
::v-deep .ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: rgba(229, 242, 255, 0.3);
}
::v-deep .ant-progress-circle .ant-progress-text {
  color: #fff;
  font-size: 12px;
}
::v-deep .ant-table-tbody .ant-table-row td {
  padding-top: 8px;
  padding-bottom: 8px;
  color: #f5f6ff;
  background: rgba(51, 85, 255, 0);
  box-shadow: 0px -1px 0px 0px rgba(51, 85, 255, 0.2);
}
::v-deep .ant-table-tbody > tr > td {
  border: none;
}
.ant-progress-circle .ant-progress-text {
  color: #ffffffa6;
}
::v-deep {
  /* 表格斑马样式 **/
  .ant-table-tbody tr:nth-child(2n) {
    background: rgba(51, 85, 255, 0.1);
  }
}

.banner {
  height: 100vh;
  width: 100%;
  background-image: url(img/KB_BG.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
}
.headers {
  position: relative;
  .headers-title {
    margin-left: 42%;
    background-image: url(img/车间生产监控大屏.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    height: 46px;
    width: 296px;
    padding-top: 1.5vh;
    text-align: center;
    font-size: 14px;
    font-family: Source Han Sans CN;
    font-weight: 400;
    color: #6bb0ff;
  }
  .title-out {
    position: absolute;
    top: 1vh;
    right: 8vh;
    font-size: 14px;
    font-family: Source Han Sans CN;
    color: #6bb0ff;
    &:hover {
      cursor: pointer;
    }
  }
  .title-out-icon {
    position: absolute;
    top: 1.5vh;
    right: 6vh;
    width: 14px;
    height: 14px;
    background-image: url(img/out.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    &:hover {
      cursor: pointer;
    }
  }
  .title-quanpin-icon {
    position: absolute;
    top: 1.5vh;
    right: 6vh;
    width: 14px;
    height: 14px;
    background-image: url(img/quanpin.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    &:hover {
      cursor: pointer;
    }
  }
}
.banner-content {
  height: 100%;
  padding: 30px 30px 44px 30px;
}
.content {
  height: 100%;
  float: left;
  display: flex;
}
.content-table {
  height: 28%;
  margin: 0 30px;
}
.work-order-list {
  width: 100% !important;
  height: 50px !important;
  margin-bottom: 100px;
  background-image: url(img/小标题.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
  font-size: 20px;
  font-family: PangMenZhengDao;
  font-weight: bold;
  color: #ffffff;
  line-height: 30px;
}
.work-order-list-1 {
  width: 100% !important;
  height: 50px !important;
  margin-bottom: 100px;
  background-image: url(img/小标题1.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
  font-size: 20px;
  font-family: PangMenZhengDao;
  font-weight: bold;
  color: #ffffff;
  line-height: 30px;
}
.menu-content {
  display: flex;
}
.left-menu-title {
  width: 100%;
  height: 50px;
  margin-bottom: 20px;
  background-image: url(img/小标题.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
  font-size: 20px;
  font-family: PangMenZhengDao;
  font-weight: bold;
  color: #ffffff;
  line-height: 42px;
}
.left-menu-content {
  flex: 1.5;
  margin-right: 2vh;
  display: flex;
  flex-direction: column;
}
.center-menu-content {
  height: 100%;
  width: 100%;
  .top {
    padding: 20px 0;
    padding-top: 20px;
    height: 12vh;
    .content-middle {
      text-align: center;
      height: calc(100% - 40px - 20px);
      color: #cad4e6ff;
    }
    .number {
      font-size: 30px;
      font-family: zcoolqingkehuangyouti;
      font-weight: bold;
      color: #fdff7d;
      margin: 0 10px;
      line-height: 10px;
    }
    .number-icon {
      display: inline-block;
      width: 137px;
      height: 91px;
      background-image: url(img/gong-dan.png);
      background-size: 100% 100%;
      background-repeat: no-repeat;
    }
    .renwu-icon {
      display: inline-block;
      width: 137px;
      height: 91px;
      background-image: url(img/renwu.png);
      background-size: 100% 100%;
      background-repeat: no-repeat;
    }
    .number-top {
      width: 14px;
      height: 14px;
      font-size: 14px;
      font-family: Source Han Sans CN;
      font-weight: bold;
      color: #fdff7d;
      line-height: 120px;
    }
  }
  .center {
    height: 30vh;
    .center-left {
      width: 30%;
      height: calc(100% - 20px);
      float: left;
      position: relative;
      .icon-l {
        width: 92px;
        height: 40vh;
        margin-top: -30px;
        background-size: 100% 100%;
        background-repeat: no-repeat;
        background-image: url(img/l.png);
      }
      .data {
        width: 80%;
        height: 100%;
        position: absolute;
        top: 0;
        right: 0;
        .content-data {
          overflow-y: auto;
          height: 30vh;
          display: flex;
          flex-direction: column;
          .middle {
            width: 100%;
            margin-bottom: 15px;
            text-align: center;
            font-size: 14px;
            font-family: Source Han Sans CN;
            font-weight: bold;
            color: #cad4e6;
            .icon-span {
              width: 6px;
              height: 10px;
              display: inline-block;
              background: #70f0ff;
              transform: skew(-40deg);
              margin-right: 10px;
              box-shadow: 0px 4px 6px 0px rgba(79, 236, 255, 0.61);
            }
            .data-number {
              font-size: 24px;
              font-family: zcoolqingkehuangyouti;
              font-weight: bold;
              color: #28edff;
            }
            .data-number-error {
              font-size: 24px;
              font-family: zcoolqingkehuangyouti;
              font-weight: bold;
              color: #fa466cff;
            }
            .data-succuss {
              width: 14px;
              height: 14px;
              font-size: 14px;
              font-family: Source Han Sans CN;
              font-weight: bold;
              color: #28edff;
            }
            .data-error {
              width: 14px;
              height: 14px;
              font-size: 14px;
              font-family: Source Han Sans CN;
              font-weight: bold;
              color: #fa466cff;
            }
          }
        }
      }
    }
    .center-right {
      width: 30%;
      height: calc(100% - 20px);
      float: right;
      position: relative;
      .icon-l {
        width: 92px;
        height: 40vh;
        margin-top: -30px;
        float: right;
        background-size: 100% 100%;
        background-repeat: no-repeat;
        background-image: url(img/r.png);
      }
      .data {
        width: 80%;
        height: 100%;
        position: absolute;
        top: 0;
        .content-data {
          overflow-y: auto;
          height: 30vh;
          display: flex;
          flex-direction: column;
          .middle {
            width: 100%;
            margin-bottom: 15px;
            text-align: center;
            font-size: 14px;
            font-family: Source Han Sans CN;
            font-weight: bold;
            color: #cad4e6;
            .icon-span {
              width: 6px;
              height: 10px;
              display: inline-block;
              background: #70f0ff;
              transform: skew(40deg);
              margin-left: 10px;
              box-shadow: 0px 4px 6px 0px rgba(79, 236, 255, 0.61);
            }
            .data-number {
              font-size: 24px;
              font-family: zcoolqingkehuangyouti;
              font-weight: bold;
              color: #28edff;
            }
            .data-number-error {
              font-size: 24px;
              font-family: zcoolqingkehuangyouti;
              font-weight: bold;
              color: #fa466cff;
            }
            .data-succuss {
              width: 14px;
              height: 14px;
              font-size: 14px;
              font-family: Source Han Sans CN;
              font-weight: bold;
              color: #28edff;
            }
            .data-error {
              width: 14px;
              height: 14px;
              font-size: 14px;
              font-family: Source Han Sans CN;
              font-weight: bold;
              color: #fa466cff;
            }
          }
        }
      }
    }
  }
  .footer {
    height: 13vh;
    overflow-y: auto;
    margin: 7vh 0 0 0;
    border-radius: 5px;
    padding: 10px;
    background: #020e1e;
    opacity: 0.5;
    .scroll-wrapper-ul {
      width: 100%;
      padding-left: 8px;
      font-family: Source Han Sans CN;
      font-weight: bold;
      color: #cad4e6;
      line-height: 120px;
    }
    .scroll-wrapper-li {
      height: 24px;
      line-height: 24px;
      display: flex;
      align-items: center;
      padding: 5px 0;
      color: #fff;
    }
    .scroll-wrapper-li > span {
      display: inline-block;
      overflow: hidden;
      text-align: left;
      font-size: 14px;
      padding: 0 12px;
      font-family: Source Han Sans CN;
      font-weight: bold;
      color: #cad4e6;
      line-height: 120px;
      // transform: scale(0.85);
      transform-origin: left center;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
    .prefix_APscC {
      display: inline-block;
      width: 7px;
      height: 7px;
      position: relative;
      // border: 3px solid #04f9fe;
      left: -7px;
      border-radius: 50%;
    }
  }
}

.ant-table-cell {
  background: rgb(8, 113, 132);
  opacity: 0.6;
}

.ant-table-cell- {
  background: rgb(5, 17, 46);
  opacity: 0.6;
  color: #fff;
}

.ant-table-thead > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-thead > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: #e6f7ff;
  opacity: 0.6;
  color: #333333;
}

h4 {
  color: #fff;
}

#mains {
  height: 30vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 3vh;
}
#mainA {
  height: 30vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 3vh;
}

#main-2 {
  height: 25vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 2vh;
}
#mainB {
  height: 25vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 2vh;
}

#main-3 {
  height: 30vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 3vh;
}
#mainC {
  height: 30vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 3vh;
}

#main-4 {
  height: 25vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 2vh;
}
#mainD {
  height: 25vh;
  margin-left: 2vh;
  margin-right: 2vh;
  padding-top: 2vh;
}

p {
  color: #fff;
  text-align: center;
  font-size: 12px;
  margin: 0px;
}
</style>
