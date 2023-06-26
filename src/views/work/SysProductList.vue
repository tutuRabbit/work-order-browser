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
      <a-button @click="handleAdd()" type="primary" icon="plus" v-has="'SysProduct:add'">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('产品定义表')" v-has="'SysProduct:export'"
        >导出
      </a-button>
      <a-button type="primary" icon="import" v-has="'SysProduct:import'" @click="importVisible = true">导入</a-button>
      <!-- <a-upload
        v-has="'SysProduct:import'"
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
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysProduct:delete'">
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
          <a @click="handleShow(record)">查看图纸</a>
          <a-divider type="vertical" />
          <a @click="handleEdit(record)" v-has="'SysProduct:edit'">编辑</a>
          <a-divider type="vertical" v-has="'SysProduct:edit'" />
          <a @click="handlePrint(record)">打印</a>
          <a-divider type="vertical" />
          <a @click="handleDetail(record)" v-has="'SysProduct:view'">详情</a>
          <a-divider type="vertical" v-has="'SysProduct:view'" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysProduct:delete'">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)" v-has="'SysProduct:view'">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysProduct:delete'">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>

    <sys-product-modal ref="modalForm" @ok="modalFormOk"></sys-product-modal>
    <print-modal ref="printForm"></print-modal>
    <!-- 导入 -->
    <j-modal title="导入产品定义" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入产品定义模板》</a></p>
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
    <import-description-modal :show.sync="importDescriptionVisible" type="product"></import-description-modal>
    <!-- 打印 -->
    <j-modal title="打印二维码" width="45%" :visible="printVisible" cancelText="关闭" @cancel="printVisible = false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24" style="width:100px;height:100px;text-align:center">
              <img id="printContent" :src="printList" width="100px" height="100px" />
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="printVisible = false">关闭</a-button>
        <a-button v-print="'#printContent'" ghost type="primary">打印</a-button>
      </template>
    </j-modal>
    <j-modal-drawing ref="showDrawing"></j-modal-drawing>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysProductModal from './modules/SysProductModal'
import printModal from './modules/printModal'
import { uploadAction, downFile } from '@/api/manage'
import { QRCodeGenerateAction } from '../../api/manage'

import ImportDescriptionModal from '@views/work/modules/importDescriptionModal'

export default {
  name: 'SysProductList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysProductModal,
    printModal,
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
      importVisible: false,
      importDescriptionVisible: false,
      fileList: [],
      queryParam: {
        filename: '产品定义'
      },
      description: '产品基础资料表管理页面',
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
          title: '产品编码',
          align: 'center',
          dataIndex: 'code',
          width: 180,
          ellipsis: true
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName',
          width: 180,
          ellipsis: true
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productNorms',
          width: 180,
          ellipsis: true
        },
        {
          title: '库存单位',
          align: 'center',
          dataIndex: 'stockPiece',
          width: 180,
          ellipsis: true
        },
        {
          title: '产品属性',
          align: 'center',
          dataIndex: 'productProperty',
          width: 180,
          ellipsis: true
        },
        {
          title: '工艺路线',
          align: 'center',
          dataIndex: 'craftRouteName',
          width: 180,
          ellipsis: true
        },
        {
          title: '最大库存',
          align: 'center',
          dataIndex: 'maxStock',
          width: 180,
          ellipsis: true
        },
        {
          title: '最小库存',
          align: 'center',
          dataIndex: 'minStock',
          width: 180,
          ellipsis: true
        },
        {
          title: '安全库存',
          align: 'center',
          dataIndex: 'securityStock',
          width: 180,
          ellipsis: true
        },
        {
          title: '库存数量',
          align: 'center',
          dataIndex: 'stockNumber',
          width: 180,
          ellipsis: true
        },
        {
          title: '创建人名称',
          align: 'center',
          dataIndex: 'userName',
          width: 180,
          ellipsis: true
        },
        {
          title: '创建时间',
          align: 'center',
          dataIndex: 'createTime',
          width: 180,
          ellipsis: true
        },
        {
          title: '在制品总数',
          align: 'center',
          dataIndex: 'totalWork',
          width: 180,
          ellipsis: true
        },
        {
          title: '在制品已入库数',
          align: 'center',
          dataIndex: 'storageQuantity',
          width: 180,
          ellipsis: true
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
        list: '/longxianda/sysProduct/list',
        delete: '/longxianda/sysProduct/delete',
        deleteBatch: '/longxianda/sysProduct/deleteBatch',
        printXls: '/longxianda/sysProduct/printXls?id=', //打印
        QRCodeGenerate: '/longxianda/sysProduct/QRCodeGenerate_product?productId=', //图片流打印
        exportXlsUrl: '/longxianda/sysProduct/exportXls',
        importExcelUrl: 'longxianda/sysProduct/importExcel', //导入接口
        download: '/attachment/tabAttachment/download?id=2' //产品定义导入模板
      },
      dictOptions: {},
      superFieldList: [],
      printVisible: false,
      printList: ''
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
    handleShow(record) {
      this.$refs.showDrawing.edit(record.code)
    },
    handlePrint(record) {
      //将图片流转换成base64
      QRCodeGenerateAction(this.url.QRCodeGenerate + record.id)
        .then(response => {
          return (
            'data:image/png;base64,' +
            btoa(new Uint8Array(response).reduce((data, byte) => data + String.fromCharCode(byte), ''))
          )
        })
        .then(data => {
          this.printList = data //data可以直接放到img标签的src中
          this.printVisible = true
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
          link.setAttribute('download', '产品定义导入模板' + Math.round(new Date().getTime()).toString())
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
      fieldList.push({ type: 'string', value: 'code', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productNorms', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockPiece', text: '库存单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productNorms', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productProperty', text: '产品属性', dictCode: '' })
      fieldList.push({ type: 'string', value: 'craftRoute', text: '工艺路线', dictCode: '' })
      fieldList.push({ type: 'int', value: 'maxStock', text: '最大库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'minStock', text: '最小库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'securityStock', text: '安全库存', dictCode: '' })
      fieldList.push({ type: 'int', value: 'stockNumber', text: '库存数量', dictCode: '' })
      fieldList.push({ type: 'string', value: 'userName', text: '创建人名称', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'createTime', text: '创建时间' })
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
