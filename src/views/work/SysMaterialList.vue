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
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'SysMaterial:add'">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('物料清单表')" v-has="'SysMaterial:export'"
        >导出
      </a-button>
      <a-button type="primary" icon="import" v-has="'SysMaterial:import'" @click="importVisible = true">导入</a-button>
      <!-- <a-upload
        v-has="'SysMaterial:import'"
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
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysMaterial:delete'">
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
          <a @click="handleEdit(record)" v-has="'SysMaterial:edit'">编辑</a>
          <a-divider type="vertical" v-has="'SysMaterial:edit'" />
          <a @click="handleDetail(record)" v-has="'SysMaterial:view'">详情</a>
          <a-divider type="vertical" v-has="'SysMaterial:view'" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysMaterial:delete'">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)"  v-has="'SysMaterial:view'">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)"  v-has="'SysMaterial:delete'">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>

    <sys-material-modal ref="modalForm" @ok="modalFormOk"></sys-material-modal>
    <!-- 导入 -->
    <j-modal title="导入物料清单" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入物料清单模板》</a></p>
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
    <import-description-modal :show.sync="importDescriptionVisible" type="material"></import-description-modal>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { uploadAction, downFile } from '@/api/manage'
import SysMaterialModal from './modules/SysMaterialModal'
import ImportDescriptionModal from '@views/work/modules/importDescriptionModal'

export default {
  name: 'SysMaterialList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysMaterialModal,
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
      description: '物料清单表管理页面',
      // 表头
      columns: [],
      //列设置
      settingColumns: [],
      defColumns: [
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
          title: '产品编码',
          align: 'center',
          dataIndex: 'lastProductCode',
          width: 180,
          ellipsis: true
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'lastProductName',
          width: 180,
          ellipsis: true
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'lastProductSpec',
          width: 180,
          ellipsis: true
        },
        {
          title: '单位',
          align: 'center',
          dataIndex: 'lastProductUnitName',
          width: 180,
          ellipsis: true
        },
        {
          title: '子产品名称',
          align: 'center',
          dataIndex: 'nextProductName',
          width: 180,
          ellipsis: true
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
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'action'
          }
        }
      ],
      url: {
        list: '/longxianda/sysMaterial/list',
        delete: '/longxianda/sysMaterial/delete',
        deleteBatch: '/longxianda/sysMaterial/deleteBatch',
        exportXlsUrl: '/longxianda/sysMaterial/exportXls',
        importExcelUrl: 'longxianda/sysMaterial/importExcel', //导入接口
        download: '/attachment/tabAttachment/download?id=6' //物料清单导入模板
      },
      dictOptions: {},
      superFieldList: []
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
      this.$refs.modalForm.title = '编辑物料清单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleAdd: function() {
      this.$refs.modalForm.add()
      this.$refs.modalForm.title = '新增物料清单'
      this.$refs.modalForm.disableSubmit = false
    },
    handleDetail: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '物料清单详情'
      this.$refs.modalForm.disableSubmit = true
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
          window.navigator.msSaveBlob(new Blob([data], { type: 'application/vnd.ms-excel' }), fileName + '.xlsx')
        } else {
          let url = window.URL.createObjectURL(new Blob([data], { type: 'application/vnd.ms-excel' }))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', '物料清单导入模板' + Math.round(new Date().getTime()).toString() + '.xlsx')
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
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      // fieldList.push({ type: 'string', value: 'lastProductId', text: '产品ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductCode', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductSpec', text: '产品规格', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'lastProductUnitId', text: '单位ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductUnitName', text: '单位', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'nextProductId', text: '子产品ID', dictCode: '' })
      fieldList.push({ type: 'string', value: 'nextProductName', text: '子产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'nextProductSpec', text: '子产品规格', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'nextProductUnitD', text: '子单位ID', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'nextProductUnitName', text: '子单位', dictCode: '' })
      fieldList.push({ type: 'int', value: 'unitQty', text: '单位用量', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'userId', text: '创建人UID', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'userName', text: '创建人名称', dictCode: '' })
      // fieldList.push({ type: 'datetime', value: 'createTime', text: '创建时间' })
      // fieldList.push({ type: 'string', value: 'orgId', text: '企业ID', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'deletedAt', text: '是否删除', dictCode: '' })
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
