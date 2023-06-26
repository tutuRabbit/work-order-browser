<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="快速查询">
              <a-input v-model="queryParam.search" placeholder="输工单编号、产品编码、产品名称、产品规格"></a-input>
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
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'SysWorkOrderList:add'">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('工单表')" v-has="'SysWorkOrderList:export'"
        >导出
      </a-button>
      <a-button type="primary" icon="import" @click="importVisible = true" v-has="'SysWorkOrderList:import'"
        >导入
      </a-button>
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysWorkOrderList:delete'">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete" />
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
      <!-- <a-button @click="print">打印</a-button> -->
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
          <a-divider type="vertical" />
          <a-popover title="自定义列" trigger="click" placement="leftBottom">
            <template slot="content">
              <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
                <a-row style="width: 400px">
                  <template v-for="(item, index) in defColumns">
                    <template v-if="item.key != 'rowIndex' && item.dataIndex != 'action'">
                      <a-col :span="12" :key="index"
                        ><a-checkbox :value="item.dataIndex"
                          ><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox
                      ></a-col>
                    </template>
                  </template>
                </a-row>
              </a-checkbox-group>
            </template>
            <a><a-icon type="setting" />设置</a>
          </a-popover>
        </span>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :components="components"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <div slot="filterDropdown">
          <a-card>
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row style="width: 400px">
                <template v-for="(item, index) in defColumns">
                  <template v-if="item.key != 'rowIndex' && item.dataIndex != 'action'">
                    <a-col :span="12" :key="index"
                      ><a-checkbox :value="item.dataIndex"
                        ><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox
                    ></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type="setting" :style="{ fontSize: '16px', color: '#108ee9' }" />

        <template slot="planAmount" slot-scope="text, record">
          <a-tooltip>
            <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left;">
              <a-progress type="circle" :width="35" :percent="item.percentage" />
              <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
              <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
            </div>
          </a-tooltip>
        </template>
        <span slot="action" slot-scope="text, record">
          <a @click="handleDetail(record)" v-has="'SysWorkOrderList:view'">查看</a>
          <a-divider type="vertical" v-has="'SysWorkOrderList:view'" />
          <a-dropdown>
            <a class="ant-dropdown-link">操作 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleShow(record)">查看图纸</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="handlePrint(record)">打印</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="workHandle(record, 'start')" v-has="'SysWorkOrderList:start'">开始</a>
              </a-menu-item>
              <a-menu-item>
                <a @click="workHandle(record, 'withdraw')" v-has="'SysWorkOrderList:recall'">撤回</a>
              </a-menu-item>
              <a-menu-item v-if="record.status != 0">
                <a @click="workHandle(record, 'finish')" v-has="'SysWorkOrderList:end'">结束</a>
              </a-menu-item>
              <a-menu-item v-if="record.status == 0">
                <a @click="handleEdit(record)" v-has="'SysWorkOrderList:edit'">编辑</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm
                  v-has="'SysWorkOrderList:delete'"
                  title="确认删除工单？操作将同时删除关联的任务和报工，删除后不可恢复"
                  @confirm="() => handleDelete(record.id)"
                >
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- 导入 -->
    <j-modal title="导入工单" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入工单模板》</a></p>
            </a-col>
            <a-col :span="24">
              <p>2.打开下载表，将对应信息填入或粘贴进本表。为保证导入成功，请使用纯文本或数字。</p>
            </a-col>
            <a-col :span="24">
              <p>3.信息输入完毕并保存后，点击下方【上传文件】按钮选择已保存的excel文档。</p>
            </a-col>
            <a-col :span="24">
              <p>4. 点击【导入】，查看 <a @click="importDescriptionVisible = true">导入说明</a></p>
            </a-col>
            <a-col :span="24">
              <a-upload
                name="file"
                :multiple="false"
                :fileList="fileList"
                :remove="handleRemove"
                :beforeUpload="beforeUpload"
              >
                <a-button type="info" icon="upload" :disabled="fileList.length != 0 ? true : false">选择文件</a-button>
              </a-upload>
            </a-col>
            <a-col :span="24"><div style="margin-top:10px">附件支持类型：XLSX，最大不能超过10M</div></a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button @click="submitImport" :disabled="fileList.length != 0 ? false : true" type="primary">导入</a-button>
      </template>
    </j-modal>
    <!--    导入说明-->
    <import-description-modal :show.sync="importDescriptionVisible" type="workOrderList"></import-description-modal>
    <sys-work-order-modal ref="modalForm" @ok="modalFormOk"></sys-work-order-modal>
    <j-modal-drawing ref="showDrawing"></j-modal-drawing>
    <!-- 打印 -->
    <a-modal title="打印" width="70%" :bodyStyle="bodyStyle" :visible="printVis" @cancel="cancel" :footer="null">
      <div style="padding:20px">
        <div class="no-print" style="text-align: right">
          <a-button v-print="'#inspectPrint'" ghost type="primary">打印</a-button>
        </div>
        <section ref="print" id="inspectPrint">
          <a-col :md="24" :sm="24">
            <div class="sign" style="text-align: left;height: inherit">
              <a-col :span="24" style="vertical-align: middle;">
                <div style="text-align: center">
                  <span style="font-size: 24px;">工单流转卡</span>
                </div>
                <div class="qrcode-div">
                  <p>工单编号:{{ this.printData.code }}</p>
                  <!-- <div style="width: 100px;height: 100px;" id="qrcode"></div> -->
                  <!-- <print-modal :id="'qrcode'" :text="codeUrl"></print-modal> -->
                  <img :src="printList" width="100px" height="100px" />
                </div>
              </a-col>
              <a-col :span="24" style="margin-top: 20px">
                <table border="1" class="table">
                  <tr>
                    <th class="ant-table-cell" style="width:10%">产品编码</th>
                    <td class="ant-table-cell">{{ this.printData.productCode }}</td>
                    <th class="ant-table-cell" style="width:10%">产品名称</th>
                    <td class="ant-table-cell">{{ this.printData.productName }}</td>
                  </tr>
                  <tr>
                    <th class="ant-table-cell" style="width:10%">计划数</th>
                    <td class="ant-table-cell">{{ this.printData.planAmount }}</td>
                    <th class="ant-table-cell" style="width:10%">产品规格</th>
                    <td class="ant-table-cell">{{ this.printData.productUnit }}</td>
                  </tr>
                  <tr>
                    <th class="ant-table-cell" style="width:10%">计划开始时间</th>
                    <td class="ant-table-cell">{{ this.printData.planStartTime }}</td>
                    <th class="ant-table-cell" style="width:10%">计划结束时间</th>
                    <td class="ant-table-cell">{{ this.printData.planEndTime }}</td>
                  </tr>
                </table>
              </a-col>
              <a-col :span="24">
                <table class="table" style="margin-top:10px">
                  <thead class="ant-table-thead" border="1">
                    <tr>
                      <th class="ant-table-cell">序号</th>
                      <th class="ant-table-cell">工序名称</th>
                      <th class="ant-table-cell">工序参数</th>
                      <th class="ant-table-cell">计划开始时间</th>
                      <th class="ant-table-cell">计划结束时间</th>
                      <th class="ant-table-cell">计划数</th>
                      <th class="ant-table-cell">合格数</th>
                      <th class="ant-table-cell">不良品数</th>
                      <th class="ant-table-cell">不良原因</th>
                      <th class="ant-table-cell">操作者</th>
                      <th class="ant-table-cell">备注</th>
                    </tr>
                  </thead>
                  <tbody class="ant-table-tbody" v-if="printDataSource.length == 0">
                    <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                      <td colspan="100" class="ant-table-cell">
                        <div class="ant-empty ant-empty-normal">
                          <div class="ant-empty-image"></div>
                          <div class="ant-empty-description">暂无数据</div>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                  <tbody class="ant-table-tbody" v-else>
                    <tr v-for="(item, index) in printDataSource" :key="index">
                      <td class="ant-table-cell">{{ index }}</td>
                      <td class="ant-table-cell">{{ item.procedureName }}</td>
                      <td class="ant-table-cell">{{ item.machiningParameter }}</td>
                      <td class="ant-table-cell">{{ item.planStartTime }}</td>
                      <td class="ant-table-cell">{{ item.planEndTime }}</td>
                      <td class="ant-table-cell">{{ item.planAmount }}</td>
                      <td class="ant-table-cell">{{ item.goodsPros }}</td>
                      <td class="ant-table-cell">{{ item.rejects }}</td>
                      <td class="ant-table-cell">{{ item.defectsStr }}</td>
                      <td class="ant-table-cell">{{ item.taskOperatorsStr }}</td>
                      <td class="ant-table-cell">{{ item.remarks }}</td>
                    </tr>
                  </tbody>
                </table>
              </a-col>
            </div>
          </a-col>
        </section>
      </div>
    </a-modal>
  </a-card>
</template>

<script>
// import printModal from './modules/printModal'
import '@/assets/less/TableExpand.less'
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import { QRCodeGenerateAction } from '../../api/manage'
import QRCode from 'qrcodejs2'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysWorkOrderModal from './modules/SysWorkOrderModal'
import { httpAction, getAction, uploadAction, downFile } from '@/api/manage'
import ImportDescriptionModal from '@views/work/modules/importDescriptionModal'

export default {
  name: 'SysWorkOrderList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    // printModal,
    VueDraggableResizable,
    ImportDescriptionModal,
    SysWorkOrderModal,
    QRCode
  },
  data() {
    this.components = {
      header: {
        cell: (h, props, children) => {
          const { key, ...restProps } = props
          const col = this.columns.find(col => {
            const k = col.dataIndex || col.key
            return k === key
          })

          if (!col || !col.width) {
            return h('th', { ...restProps }, [...children])
          }

          const dragProps = {
            key: col.dataIndex || col.key,
            class: 'table-draggable-handle',
            attrs: {
              w: 10,
              x: col.width,
              z: 1,
              axis: 'x',
              draggable: true,
              resizable: false
            },
            on: {
              dragging: (x, y) => {
                col.width = Math.max(x, 1)
              }
            }
          }
          const drag = h('vue-draggable-resizable', { ...dragProps })
          return h('th', { ...restProps, class: 'resize-table-th' }, [...children, drag])
        }
      }
    }
    return {
      // 打印
      printDataSource: [],
      codeUrl: '',
      printList: '',
      QRlink: '',
      printData: {},
      model: {},
      printVis: false,
      bodyStyle: {
        padding: '0',
        //update-begin---author:wangshuai ---date:20220104  for：[JTC-411]火狐 分屏 图片大时，与按钮重叠，样式不好------------
        // height: window.innerHeight - 240 + 'px'
        height: window.innerHeight + 'px'
        //update-begin---author:wangshuai ---date:20220104  for：[JTC-411]火狐 分屏 图片大时，与按钮重叠，样式不好------------
      },
      // end
      fileList: [],
      importVisible: false,
      importDescriptionVisible: false,
      description: '工单主表管理页面',
      //表头
      columns: [],
      //列设置
      settingColumns: [],
      defColumns: [
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
          dataIndex: 'productCode',
          width: 140,
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName',
          width: 200,
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productUnit',
          width: 140,
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount',
          width: 140,
          ellipsis: true,
          resizable: true
        },
        {
          title: '完工数',
          align: 'center',
          dataIndex: 'offNum',
          ellipsis: true,
          width: 140,
          resizable: true
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'status',
          width: 140,
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
          width: 140,
          dataIndex: 'planStartTime',
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划结束时间',
          align: 'center',
          dataIndex: 'planEndTime',
          ellipsis: true,
          width: 140,
          resizable: true
        },
        {
          title: '生产进度',
          align: 'center',
          width: 500,
          dataIndex: 'a',
          scopedSlots: { customRender: 'planAmount' },
          ellipsis: true,
          resizable: true
        },
        // {
        //   title: '生产车间',
        //   align: 'center',
        //   maxWidth: 500,
        //   dataIndex: 'manufacturingShop',
        //   ellipsis: true,
        //   resizable: true
        // },
        {
          title: '备注',
          align: 'center',
          maxWidth: 140,
          dataIndex: 'remarks',
          ellipsis: true,
          resizable: true
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'action'
          }
        }
      ],
      url: {
        list: '/longxianda/sysWorkOrder/list',
        delete: '/longxianda/sysWorkReporting/deleteWorkOrder',
        deleteBatch: '/longxianda/sysWorkReporting/deleteWorkOrderBatch',
        exportXlsUrl: '/longxianda/sysWorkOrder/exportXls',
        printXls: '/longxianda/sysWorkOrder/printXls', //打印
        QRCodeGenerateProduct: '/longxianda/sysWorkOrder/QRCodeGenerateProduct?workOrderCode=', //图片流打印
        //工单流转卡打印
        printWorkOrderCircleXls: '/longxianda/sysWorkOrder/printWorkOrderCircleXls?workOrderId=',
        importExcelUrl: 'longxianda/sysWorkOrder/importExcel', //导入
        download: '/attachment/tabAttachment/download?id=9' //工单导入模板
      },
      dictOptions: {},
      visible: false,
      productionSchedule: [],
      superFieldList: [],
      expandFieldValues: [] // 自定义字段列表
    }
  },
  created() {
    // 自定义字段
    // this.loadExpandFieldValues(1009)
    this.getSuperFieldList()
    this.initColumns()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    //列设置更改事件
    onColSettingsChange(checkedValues) {
      var key = this.$route.name + ':colsettings'
      Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
      this.settingColumns = checkedValues
      const cols = this.defColumns.filter(item => {
        if (item.key == 'rowIndex' || item.dataIndex == 'action') {
          return true
        }
        if (this.settingColumns.includes(item.dataIndex)) {
          return true
        }
        return false
      })
      this.columns = cols
    },
    initColumns() {
      //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
      var key = this.$route.name + ':colsettings'
      let colSettings = Vue.ls.get(key)
      if (colSettings == null || colSettings == undefined) {
        let allSettingColumns = []
        this.defColumns.forEach(function(item, i, array) {
          allSettingColumns.push(item.dataIndex)
        })
        this.settingColumns = allSettingColumns
        this.columns = this.defColumns
      } else {
        this.settingColumns = colSettings
        const cols = this.defColumns.filter(item => {
          if (item.key == 'rowIndex' || item.dataIndex == 'action') {
            return true
          }
          if (colSettings.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.columns = cols
      }
    },
    handleEdit: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '编辑工单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleAdd: function() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '新增工单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleDetail: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '工单详情'
      this.$refs.modalForm.disableSubmit = true
    },
    handleShow(record) {
      this.$refs.showDrawing.edit(record.productCode)
    },
    handlePrint(record) {
      var that = this
      // 后端打印
      // let url = window.location.origin + '/jeecg-boot' + this.url.printWorkOrderCircleXls + record.id
      // window.open(url)
      // 前端打印
      this.baseImg(record)
      that.printVis = true
      httpAction('/longxianda/sysWorkOrder/queryById?id=' + record.id, '', 'get').then(res => {
        if (res.success) {
          that.printData = Object.assign({}, res.result)
          that.printDataSource = that.printData.workOrderDetails
        } else {
          that.$message.warning(res.message)
        }
      })
      // setTimeout(() => {
      //   this.qrcode()
      // }, 500)
    },

    baseImg(record) {
      //将图片流转换成base64
      QRCodeGenerateAction(this.url.QRCodeGenerateProduct + record.code)
        .then(response => {
          return (
            'data:image/png;base64,' +
            btoa(new Uint8Array(response).reduce((data, byte) => data + String.fromCharCode(byte), ''))
          )
        })
        .then(data => {
          this.printList = data //data可以直接放到img标签的src中
          // this.printVisible = true
        })
    },
    //  生成二维码
    qrcode() {
      document.getElementById('qrcode').innerHTML = ''
      let that = this
      new QRCode('qrcode', {
        width: 100,
        height: 100,
        text: that.QRlink
      })
    },

    print() {
      // 打印（通过iframe进行预览，然后调用iframe进行打印）
      let doc = null
      // 获取并删除之前的节点
      let deleteIframe = document.querySelector('.printPdf')
      if (deleteIframe) {
        document.body.removeChild(deleteIframe)
      }
      getAction(this.url.printXls).then(res => {
        //流转html字符串
        const blob = new Blob([res], { type: 'text/html' })
        const reader = new FileReader()
        reader.readAsText(blob, 'utf-8')
        reader.onload = function() {
          const iframe1 = document.createElement('iframe')
          iframe1.className = 'printPdf'
          iframe1.id = 'iframe1'
          iframe1.style.display = 'none'
          document.body.appendChild(iframe1)
          doc = iframe1.contentWindow.document
          doc.write(reader.result)
          doc.close()
        }
      })
    },
    handleCancel() {
      this.importVisible = false
      this.fileList = []
    },
    cancel() {
      this.printVis = false
      this.printDataSource = []
      this.printData = {}
      // document.getElementById('qrcode').innerHTML = ''
    },
    downFiles() {
      downFile(this.url.download).then(data => {
        if (!data) {
          this.$message.warning('文件下载失败')
          return
        }
        if (typeof window.navigator.msSaveBlob !== 'undefined') {
          window.navigator.msSaveBlob(new Blob([data], { type: 'application/vnd.ms-excel' }), fileName + '.xls')
        } else {
          let url = window.URL.createObjectURL(new Blob([data], { type: 'application/vnd.ms-excel' }))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', '工单导入模板' + Math.round(new Date().getTime()).toString())
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link) //下载完成移除元素
          window.URL.revokeObjectURL(url) //释放掉blob对象
        }
      })
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      // 文件限制
      var fileNames = file.name.split('.')
      var fileType = fileNames[fileNames.length - 1].toLocaleLowerCase()
      var extList = ['xlsx', 'xls']
      if (!extList.find(item => item == fileType)) {
        this.$message.warning('只能上传.xlsx/.xls/类型的文件！')
        this.fileList = []
        return false
      }
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        if (sum > 10) {
          this.$message.warning('文件总大小超过10MB，请修改后重新上传!')
        }
      }
      return false
    },
    submitImport() {
      let pm = new FormData()
      if (this.fileList.length != 0) {
        let { fileList } = this
        fileList.forEach(file => {
          pm.append('file', file)
        })
      }
      uploadAction(this.url.importExcelUrl, pm).then(res => {
        if (res.success) {
          this.$message.success(res.message)
          this.loadData()
        } else {
          this.$message.warning(res.message)
        }
      })
      this.importVisible = false
      this.fileList = []
    },
    showModal(info) {
      // item.planAmount/item.fineAmount
      if (info.taskLists.length > 0) {
        this.visible = true
        this.productionSchedule = []
        info.taskLists.map(item => {
          let number = 0

          if (item.planAmount != null && item.fineAmount != null) {
            number = (item.fineAmount / item.planAmount) * 100
          }
          this.productionSchedule.push({
            title: item.procedureName,
            percentage: number
          })
        })
      }
    },
    hideModal() {
      this.visible = false
    },
    initDictConfig() {},
    // 工单操作
    workHandle(record, type) {
      var that = this
      that.confirmLoading = true
      var httpurl = ''
      if (type == 'start') {
        let url = '/longxianda/sysWorkOrder/queryById?id=' + record.id
        httpAction(url, '', 'get').then(ree => {
          if (ree.success) {
            let workOrderDetails = ree.result.workOrderDetails
            console.log('workOrderDetails', workOrderDetails)
            console.log('workOrderDetails.length', workOrderDetails.length)
            if (workOrderDetails.length != 0) {
              httpurl += '/longxianda/sysWorkOrder/workStart?id=' + record.id

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
            } else {
              that.$message.warning('请添加工艺路线或加工工序！')
              return false
            }
          }
        })
      } else if (type == 'withdraw') {
        httpurl += '/longxianda/sysWorkOrder/workWithdraw?id=' + record.id
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
      } else {
        httpurl += '/longxianda/sysWorkOrder/workFinish?id=' + record.id

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
      }
    },
    // 通过字典Id获取自定义字段信息
    loadExpandFieldValues(value) {
      var that = this
      let httpurl = '/longxianda/sysCustom/list?targetType=' + value
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.expandFieldValues = res.result.records
            that.expandFieldValues.map(item => {
              that.columns.splice(that.columns.length - 2, 0, {
                title: item.fieldName,
                align: 'center',
                dataIndex: item.expands.fieldValue
              })
            })
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'code', text: '工单编号', dictCode: '' })
      fieldList.push({
        type: 'select',
        value: 'status',
        text: '状态',
        options: [
          { label: '未开始', value: 0 },
          { label: '执行中', value: 10 },
          { label: '已结束', value: 100 }
        ]
      })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'planNumber', text: '计划数', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>

<style>
@import '~@assets/less/common.less';
.qrcode-div {
  position: relative;
  top: 10%;
  left: 80%;
  z-index: 999;
}
.pane {
  padding: 20px;
}
.table {
  table-layout: auto;
  width: 100%;
}
.ant-table-cell {
  border: 1px solid #ddd;
}
.ant-table-thead > tr > th,
.ant-table-tbody > tr > td {
  padding: 10px;
}
.ant-table-thead > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-thead > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: #ffffff;
  opacity: 0.6;
  color: #333333;
}

.table-draggable-handle {
  height: 100% !important;
  left: auto !important;
  right: -5px;
  cursor: col-resize;
  touch-action: none;
  border: none;
  position: absolute;
  transform: none !important;
  bottom: 0;
}
.resize-table-th {
  position: relative;
}

.spanC {
  color: #108ee9;
}
</style>
