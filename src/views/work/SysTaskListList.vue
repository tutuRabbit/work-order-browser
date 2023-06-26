<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="快速查询">
              <a-input v-model="queryParam.search" placeholder="输产品名称、产品编码、产品规格"></a-input>
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
      <!--<a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
      <a-button type="primary" icon="download" @click="handleExportXls('任务表')" v-has="'SysTaskListList:export'"
        >导出</a-button
      >
      <!-- <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
        v-has="'SysTaskListList:import'"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysTaskListList:delete'">
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

        <!-- 分配列表 -->
        <template slot="name" slot-scope="text, record">
          <a-button @click="listData(record)">{{
            record.taskOperatorsStr ? record.taskOperatorsStr : '分配'
          }}</a-button>
        </template>
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
          <a @click="handleEdit(record)" v-has="'SysTaskListList:edit'">编辑</a>
          <a-divider type="vertical" v-has="'SysTaskListList:edit'" />
          <a @click="handleDetail(record)" v-has="'SysTaskListList:view'">详情</a>
          <a-divider type="vertical" v-has="'SysTaskListList:view'" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysTaskListList:delete'">
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

    <!-- table区域-begin -->
    <!--<div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <j-vxe-table
        ref="vTable"
        toolbar
        row-number
        row-selection
        drag-sort
        keep-source
        :height="580"
        :loading="loading"
        :dataSource="dataSource"
        :columns="columns1"
        style="margin-top: 8px;"
        @edit-closed="handleEditClosed"
        @cell-click="clickCell"
      >
        <span slot="action" slot-scope="props">
          <a @click="handleEdit(props.row)">编辑</a>
          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多<a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(props.row)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(props.row.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </j-vxe-table>
    </div>-->

    <j-modal title="分配列表" width="45%" :visible="listVisible" cancelText="关闭" @cancel="listVisible = false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="分配列表">
                <a-select
                  style="width: 100%"
                  mode="multiple"
                  placeholder="请选择"
                  v-model="listParam.taskOperators"
                  :allowClear="false"
                  :showSearch="true"
                  optionFilterProp="label"
                >
                  <a-select-option
                    v-for="(item, index) in distributionList"
                    :key="index"
                    :value="item.value"
                    :label="item.title"
                    :disabled="item.disabled"
                    >{{ item.title }}</a-select-option
                  >
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
    <sys-task-list-modal ref="modalForm" @ok="modalFormOk"></sys-task-list-modal>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { filterObj } from '@/utils/util'
import SysTaskListModal from './modules/SysTaskListModal'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { httpAction, getAction } from '@/api/manage'

export default {
  name: 'SysTaskListList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysTaskListModal,
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
      listParam: {},
      taskOperatorsList: '',
      listLoading: false,
      distributionList: [], // 分配列表
      listVisible: false,
      description: '工序任务表管理页面',
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
          width: 200,
          dataIndex: 'projectCode'
        },
        {
          title: '产品编号',
          align: 'center',
          width: 200,
          dataIndex: 'productCode'
        },
        {
          title: '产品名称',
          align: 'center',
          width: 200,
          dataIndex: 'productName'
        },
        {
          title: '产品规格',
          align: 'center',
          width: 200,
          dataIndex: 'productSpec'
        },
        {
          title: '工序编号',
          align: 'center',
          width: 200,
          dataIndex: 'procedureCode'
        },
        {
          title: '工序名称',
          align: 'center',
          width: 200,
          dataIndex: 'procedureName'
        },
        {
          title: '加工参数',
          align: 'center',
          dataIndex: 'machiningParameter',
          width: 180,
          ellipsis: true
        },
        {
          title: '工序状态',
          align: 'center',
          dataIndex: 'procedureStatus',
          width: 200,
          customRender: function(t, r, index) {
            if (r.procedureStatus == 0) {
              return '未开始'
            } else if (r.procedureStatus == 10) {
              return '执行中'
            } else if (r.procedureStatus == 100) {
              return '已结束'
            } else if (r.procedureStatus == 99) {
              return '已取消'
            } else {
              return '-'
            }
          }
        },
        {
          title: '计划开始时间',
          align: 'center',
          width: 200,
          dataIndex: 'procedureStartTime'
        },
        {
          title: '计划结束时间',
          align: 'center',
          width: 200,
          dataIndex: 'procedureEndTime'
        },
        {
          title: '实际开始时间',
          align: 'center',
          width: 200,
          dataIndex: 'procedureStartRealTime'
        },
        {
          title: '实际结束时间',
          align: 'center',
          width: 200,
          dataIndex: 'procedureEndRealTime'
        },
        {
          title: '报工权限',
          align: 'center',
          width: 200,
          dataIndex: 'outputObject'
        },
        {
          title: '分配列表',
          align: 'center',
          width: 200,
          dataIndex: 'taskOperatorsStr',
          scopedSlots: { customRender: 'name' }
        },
        {
          title: '生产车间',
          align: 'center',
          width: 200,
          dataIndex: 'manufacturingShopStr'
        },
        {
          title: '报工配比',
          align: 'center',
          dataIndex: 'outputRate'
        },
        {
          title: '计划数',
          align: 'center',
          width: 200,
          dataIndex: 'planAmount'
        },
        {
          title: '不良品数',
          align: 'center',
          width: 200,
          dataIndex: 'defectsAmount'
        },
        {
          title: '良品数',
          align: 'center',
          width: 200,
          dataIndex: 'fineAmount'
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
      // 表头
      columns1: [
        {
          title: '工单编号',
          key: 'projectCode',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '产品编号',
          align: 'center',
          key: 'productCode',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '产品名称',
          align: 'center',
          key: 'productName',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '产品规格',
          align: 'center',
          key: 'productSpec',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '工序编号',
          align: 'center',
          key: 'procedureCode',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '工序名称',
          align: 'center',
          key: 'procedureName',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '工序状态',
          align: 'center',
          key: 'procedureStatus',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '计划开始时间',
          align: 'center',
          key: 'procedureStartTime',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '计划结束时间',
          align: 'center',
          key: 'procedureEndTime',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '实际开始时间',
          align: 'center',
          key: 'procedureStartRealTime',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '实际结束时间',
          align: 'center',
          key: 'procedureEndRealTime',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '报工权限',
          align: 'center',
          key: 'outputObject',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '分配列表',
          key: 'taskOperatorsStr',
          type: JVXETypes.selectMultiple,
          width: '180px',
          // 下拉选项
          options: [],
          defaultValue: '3d464b4ea0d2491aab8a7bde74c57e95,a75d45a015c44384a04449ee80dc3503',
          allowInput: true,
          placeholder: '请选择'
        },
        {
          title: '报工配比',
          align: 'center',
          key: 'outputRate',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '计划数',
          align: 'center',
          key: 'planAmount',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '不良品数',
          align: 'center',
          key: 'defectsAmount',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '良品数',
          align: 'center',
          key: 'fineAmount',
          type: JVXETypes.normal,
          width: '80px',
          defaultValue: ''
        },
        {
          title: '操作',
          align: 'center',
          key: 'action',
          type: JVXETypes.slot,
          fixed: 'right',
          minWidth: '140px',
          align: 'center',
          slotName: 'action'
        }
      ],
      url: {
        list: '/longxianda/sysTaskList/list',
        delete: '/longxianda/sysWorkReporting/deleteTaskList',
        deleteBatch: '/longxianda/sysWorkReporting/deleteTaskListBatch',
        exportXlsUrl: '/longxianda/sysTaskList/exportXls',
        importExcelUrl: 'longxianda/sysTaskList/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      options: [],
      newid: 0
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
    handleEdit: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '编辑任务'
      this.$refs.modalForm.disableSubmit = false
    },
    handleAdd: function() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '新增任务'
      this.$refs.modalForm.disableSubmit = false
    },
    handleDetail: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '任务详情'
      this.$refs.modalForm.disableSubmit = true
    },
    // 分配列表
    submit() {
      const that = this
      this.listVisible = false
      that.listParam.taskOperators = that.listParam.taskOperators.join(',')
      httpAction('/longxianda/sysTaskList/edit', that.listParam, 'put').then(res => {
        if (res.success) {
          that.$message.success(res.message)
          this.listParam = {}
          this.loadData()
        } else {
          that.$message.warning(res.message)
        }
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
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.filters)
      param.field = this.getQueryField()
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
    },
    getQueryField() {
      //TODO 字段权限控制
      var str = 'id,'
      this.columns.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    listData(val) {
      var that = this
      let alist = []
      this.listParam = Object.assign({}, val)
      // console.log('this.listParam.taskOperators', this.listParam.taskOperators)
      if (this.listParam.taskOperators) {
        this.listParam.taskOperators = this.listParam.taskOperators.split(',')
      } else {
        this.listParam.taskOperators = []
      }
      let httpurl = '/longxianda/sysProcedure/getUserByProcedureId?procedureId=' + that.listParam.procedureId
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm).then(res => {
        console.log('resValue', res)
        if (res.length > 0) {
          res.map(item => {
            alist.push({
              title: item.realname,
              value: item.id,
              disabled: true
            })
          })
          console.log('alist', alist)
          console.log('this.listParam.procedureStatus', this.listParam.procedureStatus)
          if (this.listParam.taskOperators !== 0) {
            let arr = this.listParam.taskOperators
            alist.map(item => {
              item.disabled = false
              arr.map(val => {
                if (val == item.value) {
                  item.disabled = true
                }
              })
            })
          }
          console.log('alist', alist)
          that.distributionList = alist
        } else {
          that.$message.warning(res.message)
        }
      })
      this.listVisible = true
      console.log(' this.listParam.taskOperators', this.listParam.taskOperators)
      console.log('  that.distributionList', that.distributionList)
    },
    // 分配列表end
    initDictConfig() {},
    getDistributionList(procedureId) {
      var that = this
      var alist = []
      let httpurl = '/longxianda/sysProcedure/getUserByProcedureId?procedureId=' + procedureId
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.length > 0) {
            res.map(item => {
              alist.push({
                title: item.realname,
                value: item.id
              })
            })
            if (alist.toSting != that.columns1[12].options.toSting) {
              that.columns1[12].options = alist
            }
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 点击项
    clickCell(event) {
      let info = event.row
      //this.columns1[12].options = []
      this.getDistributionList(info.procedureId)
    },
    // 单元格编辑完成之后触发的事件
    handleEditClosed(event) {
      let { $table, row, column } = event

      let field = column.property
      let cellValue = row[field]
      // 判断单元格值是否被修改
      if ($table.isUpdateByRow(row, field)) {
        // 校验当前行
        $table.validate(row).then(errMap => {
          // 校验通过
          if (!errMap) {
            // 【模拟保存】
            let hideLoading = this.$message.loading(`正在保存"${column.title}"`, 0)
            var that = this
            let httpurl = '/longxianda/sysTaskList/edit'
            httpAction(httpurl, { id: row.id, taskOperators: cellValue }, 'post')
              .then(res => {
                if (res.code == 200) {
                  that.$message.success(`"${column.title}"保存成功！`)
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                hideLoading()
                that.confirmLoading = false
              })
          }
        })
      }
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'projectCode', text: '工单编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productSpec', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'processId', text: '工序ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'procedureCode', text: '工序编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'procedureName', text: '工序名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'machiningParameter', text: '加工参数', dictCode: '' })
      fieldList.push({ type: 'int', value: 'procedureStatus', text: '', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'procedureStartTime', text: '计划开始时间' })
      fieldList.push({ type: 'datetime', value: 'procedureEndTime', text: '计划结束时间' })
      fieldList.push({ type: 'datetime', value: 'procedureStartRealTime', text: '实际开始时间' })
      fieldList.push({ type: 'datetime', value: 'procedureEndRealTime', text: '实际结束时间' })
      fieldList.push({ type: 'string', value: 'outputObject', text: '报工权限', dictCode: '' })
      fieldList.push({ type: 'string', value: 'outputRate', text: '报工配比', dictCode: '' })
      fieldList.push({ type: 'int', value: 'planAmount', text: '计划数', dictCode: '' })
      fieldList.push({ type: 'int', value: 'defectsAmount', text: '不良品数', dictCode: '' })
      fieldList.push({ type: 'int', value: 'fineAmount', text: '良品数', dictCode: '' })
      fieldList.push({ type: 'int', value: 'distributeState', text: '是否分配', dictCode: '' })
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
