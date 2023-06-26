<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="单据编号">
              <a-input placeholder="请输入单据编号" v-model="queryParam.code" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="合同编号">
              <a-input placeholder="请输入" v-model="queryParam.contractCode" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="客户名称">
              <a-input placeholder="请输入" v-model="queryParam.customName" />
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
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'SalesManagement:add'">创建生产订单</a-button>
      <!-- <a-button
        type="primary"
        icon="download"
        @click="handleExportXls('出入库明细表2')"
        v-has="'SalesManagement:export'"
        >导出</a-button
      >
      <a-button type="primary" icon="import" @click="importVisible = true" v-has="'SalesManagement:import'"
        >导入</a-button
      > -->
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SalesManagement:delete'">
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
        rowKey="saleOrderId"
        :columns="columns"
        :components="components"
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

        <template slot="productRate" slot-scope="text, record">
          <a-progress :percent="record.productRate" status="active" />
        </template>
        <template slot="orderRate" slot-scope="text, record">
          <a-progress :percent="record.orderRate" status="active" />
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
          <a @click="handleEdit(record)" v-has="'SalesManagement:edit'">编辑</a>
          <a-divider type="vertical" v-has="'SalesManagement:edit'" />
          <a @click="handleDetail(record)" v-has="'SalesManagement:view'">详情</a>
          <a-divider type="vertical" v-has="'SalesManagement:view'" />
          <a-popconfirm
            title="确认删除销售订单？操作将同时删除关联的工单、任务和报工，删除后不可恢复"
            @confirm="() => handleDelete(record.saleOrderId)"
            v-has="'SalesManagement:delete'"
          >
            <a>删除</a>
          </a-popconfirm>
        </span>
      </a-table>
    </div>
    <!-- 导入 -->
    <j-modal title="导入销售订单" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入销售订单模板》</a></p>
            </a-col>
            <a-col :span="24">
              <p>2.打开下载表，将对应信息填入或粘贴进本表。为保证导入成功，请使用纯文本或数字。</p>
            </a-col>
            <a-col :span="24">
              <p>3.信息输入完毕并保存后，点击下方【上传文件】按钮选择已保存的excel文档。</p>
            </a-col>
            <a-col :span="24">
              <p>4. 点击【导入】</p>
            </a-col>
            <a-col :span="4">
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
            <a-col :span="20">附件支持类型：XLSX，最大不能超过10M</a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button @click="submitImport" :disabled="fileList.length != 0 ? false : true" type="primary">导入</a-button>
      </template>
    </j-modal>
    <sales-modal ref="modalForm" @ok="modalFormOk"></sales-modal>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SalesModal from './modules/SalesModal'
import { httpAction, deleteAction, uploadAction, downFile, getAction } from '@/api/manage'

export default {
  name: 'SalesManagement',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SalesModal,
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
      description: '  管理页面',
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
          title: '单据编号',
          align: 'center',
          width: 180,
          dataIndex: 'code'
        },
        {
          title: '合同编号',
          align: 'center',
          width: 180,
          dataIndex: 'contractCode'
        },
        {
          title: '客户名称',
          align: 'center',
          width: 180,
          dataIndex: 'customName'
        },
        {
          title: '工单数',
          align: 'center',
          width: 120,
          dataIndex: 'workOrderNum'
        },
        {
          title: '已结束订单',
          align: 'center',
          width: 120,
          dataIndex: 'offWorkOrderNum'
        },
        {
          title: '计划数',
          align: 'center',
          width: 120,
          dataIndex: 'planNum'
        },
        {
          title: '完工数',
          align: 'center',
          width: 120,
          dataIndex: 'offNum'
        },
        {
          title: '生产进度(%)',
          align: 'center',
          width: 200,
          dataIndex: 'productRate',
          scopedSlots: { customRender: 'productRate' }
        },
        {
          title: '单据进度(%)',
          align: 'center',
          width: 200,
          dataIndex: 'orderRate',
          scopedSlots: { customRender: 'orderRate' }
        },
        {
          title: '重要等级',
          align: 'center',
          width: 120,
          dataIndex: 'importantGradeName'
        },
        {
          title: '订单性质',
          align: 'center',
          width: 120,
          dataIndex: 'orderNatureName'
        },
        {
          title: '生产车间',
          align: 'center',
          width: 160,
          dataIndex: 'manufacturingShop'
        },
        {
          title: '创建时间',
          align: 'center',
          width: 140,
          dataIndex: 'createTime'
        },
        // {
        //   title: '最后更新时间',
        //   align: 'center',
        //   dataIndex: 'updateTime'
        // },
        {
          title: '创建人',
          align: 'center',
          width: 120,
          dataIndex: 'userName'
        },
        // {
        //   title: '备注',
        //   align: 'center',
        //   maxWidth: 500,
        //   ellipsis: true,
        //   dataIndex: 'remarks'
        // },
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
        list: '/longxianda/sysSaleOrder/list',
        delete: '/longxianda/sysSaleOrder/delete',
        deleteBatch: '/longxianda/sysSaleOrder/deleteBatch',
        exportXlsUrl: '/longxianda/sysSaleOrder/exportXls',
        importExcelUrl: 'longxianda/sysSaleOrder/importExcel', //导入
        download: '/attachment/tabAttachment/download?id=4' //导入模板
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
      this.$refs.modalForm.title = '编辑生产订单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleAdd: function() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '创建销售订单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleDetail: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '生产订单详情'
      this.$refs.modalForm.disableSubmit = true
    },
    handleDelete(id) {
      if (!this.url.delete) {
        this.$message.error('请设置url.delete属性!')
        return
      }
      var that = this
      deleteAction(that.url.delete + '?id=' + id).then(res => {
        if (res.success) {
          //重新计算分页问题
          that.reCalculatePage(1)
          that.$message.success(res.message)
          that.loadData()
        } else {
          that.$message.warning(res.message)
        }
      })
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
          link.setAttribute('download', '不良品导入模板' + Math.round(new Date().getTime()).toString())
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
      fieldList.push({ type: 'string', value: 'code', text: '单号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'status', text: '状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productId', text: '产品ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productCode', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'planNumber', text: '计划数', dictCode: '' })
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

.spanC {
  color: #108ee9;
}

.spanC :last-child {
  display: none;
}
</style>
