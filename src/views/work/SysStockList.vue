<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="快速查询">
              <a-input v-model="queryParam.search" placeholder="产品编码、产品名称 、产品规格"></a-input>
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
      <!-- <a-button style="display:none;" @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('库存余额表')" v-has="'SysStockList：export'"
        >导出</a-button
      >
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
      <!-- <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown> -->
    </div>

    <!-- table区域-begin -->
    <div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

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
          <a @click="handleDetail(record)" v-has="'SysStockList：view'">详情</a>
          <a-divider type="vertical" v-has="'SysStockList：view'" />
          <a @click="goLibraryDetailed(record)" v-has="'SysStockList：goLibrary'">收发明细</a>
          <!-- <a-divider type="vertical" />
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" /> -->
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

    <sys-stock-modal ref="modalForm" @ok="modalFormOk"></sys-stock-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysStockModal from './modules/SysStockModal'
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'

export default {
  name: 'SysStockList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysStockModal,
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
      description: '库存余额表',
      queryParam: {
        filename: '库存余额'
      },
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
          title: '库位',
          align: 'center',
          dataIndex: 'storageLocationNames',
          width: 180,
          ellipsis: true
        },
        {
          title: '待入库数',
          align: 'center',
          dataIndex: 'stockPending',
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
        exportXlsUrl: '/longxianda/sysProduct/exportXls',
        importExcelUrl: 'longxianda/sysProduct/importExcel'
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
    initDictConfig() {},
    goLibraryDetailed(record) {
      this.$router.push({ path: '/iswork/StockBookList', query: { productCode: record.code } })
    },
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
