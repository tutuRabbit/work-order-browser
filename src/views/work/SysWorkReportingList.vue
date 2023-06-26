<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"></a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'SysWorkReportingList:add'">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('报工表')" v-has="'SysWorkReportingList:export'"
        >导出
      </a-button>
      <a-button type="primary" icon="import" @click="importVisible = true" v-has="'SysWorkReportingList:import'"
        >导入
      </a-button>
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
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysWorkReportingList:delete'">
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

        <!-- <template slot="name" slot-scope="text, record">
          <a-button @click="listData(record)">{{ record.operatorName }}</a-button>
        </template> -->
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)" v-has="'SysWorkReportingList:edit'">编辑</a>

          <a-divider type="vertical" v-has="'SysWorkReportingList:edit'" />
          <a @click="handleDetail(record)" v-has="'SysWorkReportingList:view'">详情</a>
          <a-divider type="vertical" v-has="'SysWorkReportingList:view'" />
          <a-popconfirm
            title="确定删除吗?"
            @confirm="() => handleDelete(record.id)"
            v-has="'SysWorkReportingList:delete'"
          >
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)" v-has="'SysWorkReportingList:view'">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysWorkReportingList:delete'">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <!-- 分配列表 -->
    <j-modal title="分配列表" width="45%" :visible="listVisible" cancelText="关闭" @cancel="listVisible = false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="分配列表">
                <a-select defaultValue="请选择" placeholder="请选择" v-model="listParam.operatorName">
                  <a-select-option
                    v-for="(item, index) in ProducerList"
                    :key="index"
                    @click="ProducerChange(item)"
                    :value="item.realname"
                    >{{ item.realname }}
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button type="primary" @click="submit">确定</a-button>
        <a-button :loading="listLoading" @click="listVisible = false">关闭</a-button>
      </template>
    </j-modal>
    <!-- 导入 -->
    <j-modal title="导入报工" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入报工模板》</a></p>
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
    <import-description-modal :show.sync="importDescriptionVisible" type="workReporting"></import-description-modal>
    <sys-work-reporting-modal ref="modalForm" @ok="modalFormOk"></sys-work-reporting-modal>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import { postAction } from '@/api/manage'
import { mixinDevice } from '@/utils/mixin'
import { httpAction, getAction } from '@/api/manage'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysWorkReportingModal from './modules/SysWorkReportingModal'
import { uploadAction, downFile } from '@/api/manage'
import ImportDescriptionModal from '@views/work/modules/importDescriptionModal'

export default {
  name: 'SysWorkReportingList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysWorkReportingModal,
    ImportDescriptionModal,
    VueDraggableResizable
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
      fileList: [],
      importVisible: false,
      importDescriptionVisible: false,
      description: '报工管理页面',
      // 表头
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
          width: 120,
          dataIndex: 'projectCode'
        },
        {
          title: '产品编号',
          align: 'center',
          width: 120,
          dataIndex: 'productCode'
        },
        {
          title: '产品名称',
          align: 'center',
          width: 120,
          dataIndex: 'productName'
        },
        {
          title: '产品规格',
          align: 'center',
          width: 120,
          dataIndex: 'productSpec'
        },
        // {
        //   title: '生产人名称',
        //   width: 180,
        //   align: 'center',
        //   dataIndex: 'operatorName'
        // },
        {
          title: '开始时间',
          align: 'center',
          width: 120,
          dataIndex: 'planStartTime'
        },
        {
          title: '结束时间',
          align: 'center',
          width: 120,
          dataIndex: 'planEndTime'
        },
        {
          title: '工序编号',
          align: 'center',
          width: 120,
          dataIndex: 'processCode'
        },
        {
          title: '工序名称',
          align: 'center',
          width: 120,
          dataIndex: 'processName'
        },
        {
          title: '工序状态',
          align: 'center',
          dataIndex: 'processStatus',
          width: 120,
          customRender: function(t, r, index) {
            if (r.processStatus == 0 || r.processStatus == 10) {
              return '执行中'
            } else if (r.processStatus == 100) {
              return '已结束'
            } else if (r.processStatus == 99) {
              return '已取消'
            } else {
              return '-'
            }
          }
        },
        {
          title: '生产人名称',
          align: 'center',
          width: 120,
          dataIndex: 'operatorName'
        },
        {
          title: '生产车间',
          align: 'center',
          width: 120,
          dataIndex: 'manufacturingShop'
        },
        {
          title: '报工数',
          align: 'center',
          width: 120,
          dataIndex: 'outputNumber'
        },
        {
          title: '良品数',
          align: 'center',
          width: 120,
          dataIndex: 'fineNumber'
        },
        {
          title: '不良品数',
          align: 'center',
          width: 120,
          dataIndex: 'defectsNumber'
        },
        {
          title: '不良品项',
          align: 'center',
          width: 120,
          dataIndex: 'defectsStr'
        },
        {
          title: '报工时长（分钟）',
          align: 'center',
          width: 120,
          dataIndex: 'outputDuration'
        },
        {
          title: '计价类型',
          align: 'center',
          dataIndex: 'pricingType',
          width: 120,
          customRender: function(t, r, index) {
            if (r.pricingType == '1') {
              return '计时'
            } else if (r.pricingType == '2') {
              return '计件'
            }
          }
        },
        {
          title: '工资单价',
          align: 'center',
          width: 120,
          dataIndex: 'unitPrice'
        },
        {
          title: '预计工资',
          align: 'center',
          width: 120,
          dataIndex: 'expectedSalary'
        },
        {
          title: '审批人',
          align: 'center',
          width: 120,
          dataIndex: 'approve'
        },
        {
          title: '审批状态',
          align: 'center',
          width: 120,
          dataIndex: 'approveStatus',
          customRender: function(t, r, index) {
            if (r.approveStatus == 0) {
              return '未审批'
            } else if (r.approveStatus == 1) {
              return '已审批'
            } else {
              return '-'
            }
          }
        },
        {
          title: '创建时间',
          align: 'center',
          width: 120,
          dataIndex: 'createTime'
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
        list: '/longxianda/sysWorkReporting/list',
        delete: '/longxianda/sysWorkReporting/delete',
        deleteBatch: '/longxianda/sysWorkReporting/deleteBatch',
        exportXlsUrl: '/longxianda/sysWorkReporting/exportXls',
        importExcelUrl: 'longxianda/sysWorkReporting/importExcel', //导入
        download: '/attachment/tabAttachment/download?id=8' //报工导入模板
      },
      dictOptions: {},
      listParam: {},
      listLoading: false,
      listVisible: false,
      ProducerList: null, // 生产人列表
      superFieldList: [],
      listCont: []
    }
  },
  created() {
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
    handleCancel() {
      this.importVisible = false
      this.fileList = []
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
          link.setAttribute('download', '报工导入模板' + Math.round(new Date().getTime()).toString())
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
    // 获取生产人列表
    loadProducerList(id) {
      var that = this
      let httpurl = '/longxianda/sysProcedure/queryUsersByProcedure?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res) {
            that.ProducerList = res
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 生产人响应监听
    ProducerChange(info) {
      this.listParam.operatorId = info.id // 生产人ID
    },
    listData(val) {
      console.log('val', val)
      this.getInfo(val.id)
      // this.listParam.operatorName = val.operatorName
      // this.listParam.operatorId = val.operatorId
      // this.listParam.id = val.id
      this.listVisible = true
    },
    submit() {
      if (this.listParam.processStatus == 0) {
        this.listParam.processStatus = 10
      }
      // 报工时长转小数类型
      this.listParam.outputDuration = parseFloat(this.listParam.outputDuration)
      // 不良品项列表转json
      this.listParam.defects = this.listCont
      postAction('/longxianda/sysWorkReporting/save', this.listParam)
        .then(res => {
          this.listLoading = true
          if (res.success) {
            this.$message.success(res.message)
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.listVisible = false
          this.listLoading = false
          this.listParam = {}
          this.loadData()
        })
    },
    // 编辑时,通过id获取详情
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysWorkReporting/queryById?id=' + id
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            that.listParam = Object.assign({}, res.result)
            that.listParam.defects = res.result.defects
            // 获取工序任务列表
            // that.loadProcedureList(res.result.projectId)
            // 获取工序任务信息
            that.loadProcedureInfo(res.result.taskListId)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 通过工序id,获取工序列表
    loadDefectsList(id) {
      var that = this
      let httpurl = '/longxianda/sysProcedure/queryById?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.defectsList = JSON.parse(res.result.defects)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取工序任务列表
    loadProcedureList(id) {
      var that = this
      let httpurl = '/longxianda/sysTaskList/list?projectId=' + id
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            that.ProcedureList = res.result.records
            console.log('  that.ProcedureList', that.ProcedureList)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 通过工序任务id,获取工序任务详情
    loadProcedureInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysTaskList/queryById?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            // that.ProcedureChange(res.result)
            that.loadProducerList(res.result.procedureId)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 工序任务响应监听
    ProcedureChange(info) {
      // 获取生产人列表
      this.loadProducerList(info.procedureId)
      // 通过工序id获取工序信息,得到工序不良品项
      // this.loadDefectsList(info.procedureId)
      // 获取绩效信息
      // this.loadMeritPayList(info.productCode, info.procedureCode)
      this.listParam.processStatus = info.procedureStatus // 工序状态
      this.listParam.processCode = info.procedureCode // 工序编号
      this.listParam.taskListId = info.id // 工序任务ID
      this.planAmount = info.planAmount // 计划数
      this.outputRate = info.outputRate // 报工配比
      this.fineAmount = info.fineAmount // 实际数
      this.kebaoStr = info.fineAmount + '/' + info.planAmount
      //                    实际进度                    总进度
    },
    handleChange(value, key, column) {
      // console.log('column', column)
      // console.log('key', key)
      // console.log('value', value)
      const newData = [...this.dataSource]
      const target = newData.filter(item => key === item.key)[0]
      if (target) {
        target[column] = value
        this.dataSource = newData
      }
    },
    edit(key) {
      const newData = [...this.dataSource]
      const target = newData.filter(item => key === item.key)[0]
      this.editingKey = key
      if (target) {
        target.editable = true
        this.dataSource = newData
      }
    },
    cancel(key) {
      const newData = [...this.dataSource]
      const target = newData.filter(item => key === item.key)[0]
      this.editingKey = ''
      if (target) {
        Object.assign(target, this.cacheData.filter(item => key === item.key)[0])
        delete target.editable
        this.dataSource = newData
      }
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'projectCode', text: '工单编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productSpec', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'planStartTime', text: '开始时间' })
      fieldList.push({ type: 'datetime', value: 'planEntTime', text: '结束时间' })
      fieldList.push({ type: 'string', value: 'processCode', text: '工序编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processName', text: '工序名称', dictCode: '' })
      fieldList.push({ type: 'int', value: 'processStatus', text: '工序状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'operatorId', text: '生产人ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'operatorName', text: '生产人名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outputNumber', text: '报工数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fineNumber', text: '良品数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'defectsNumber', text: '不良品数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'defects', text: '不良品项', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processRealNumber', text: '工序实际数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processPlanNumber', text: '工序计划数', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outputDuration', text: '报工时长', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outputId', text: '绩效', dictCode: '' })
      fieldList.push({ type: 'string', value: 'pricingType', text: '计价类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'unitPrice', text: '工资单价', dictCode: '' })
      fieldList.push({ type: 'string', value: 'expectedSalary', text: '预计工资', dictCode: '' })
      fieldList.push({ type: 'string', value: 'standardOutput', text: '标准产出', dictCode: '' })
      fieldList.push({ type: 'string', value: 'standardTaskTime', text: '标准工时', dictCode: '' })
      fieldList.push({ type: 'string', value: 'expectStatus', text: '预计状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'approve', text: '审批人', dictCode: '' })
      fieldList.push({ type: 'int', value: 'approveStatus', text: '审批状态', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'createTime', text: '创建时间' })
      fieldList.push({ type: 'string', value: 'orgId', text: '企业ID', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style>
@import '~@assets/less/common.less';
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
</style>
