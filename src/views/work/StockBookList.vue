<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="单号">
              <a-input placeholder="请输入单号" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="产品编码">
              <a-input placeholder="请输入产品编码" v-model="queryParam.productCode"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="产品名称">
                <a-input placeholder="请输入产品名称" v-model="queryParam.productName"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('库存收发明细')" v-has="'StockBookList：export'"
        >导出
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
      <!-- <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown> -->
    </div>
    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <a style="margin-left: 24px"></a>
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
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :components="components"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
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
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <stockBook-modal ref="modalForm" @ok="modalFormOk"></stockBook-modal>
  </a-card>
</template>

<script>
import StockBookModal from './modules/StockBookModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import Vue from 'vue'
import VueDraggableResizable from 'vue-draggable-resizable'
export default {
  name: 'StockBookList',
  mixins: [JeecgListMixin],
  components: {
    StockBookModal,
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
      description: '库存收发明细管理页面',
      //表头
      columns: [],
      //列设置
      settingColumns: [],
      //列定义
      defColumns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          },
          ellipsis: true
        },
        {
          title: '产品编码',
          align: 'center',
          width: 200,
          dataIndex: 'productCode',
          ellipsis: true
        },
        {
          title: '产品名称',
          align: 'center',
          width: 200,
          dataIndex: 'productName',
          ellipsis: true
        },
        {
          title: '产品规格',
          align: 'center',
          width: 200,
          dataIndex: 'productSpec',
          ellipsis: true
        },
        {
          title: '库存单位',
          align: 'center',
          width: 200,
          dataIndex: 'productUnitName',
          ellipsis: true
        },
        {
          title: '单据数量',
          align: 'center',
          width: 200,
          dataIndex: 'sheetNumber',
          ellipsis: true
        },
        {
          title: '库存变更数量',
          align: 'center',
          width: 200,
          dataIndex: 'changeNumber',
          ellipsis: true
        },
        {
          title: '当前库存数量',
          align: 'center',
          width: 200,
          dataIndex: 'currentStockNumber',
          ellipsis: true
        },
        {
          title: '库存收发时间',
          align: 'center',
          width: 200,
          dataIndex: 'stockTime',
          ellipsis: true
        },
        {
          title: '单据类型',
          align: 'center',
          dataIndex: 'type',
          width: 200,
          customRender: (text, record, index) => {
            let re = ''
            if (text === '1') {
              re = '入库单'
            } else if (text === '2') {
              re = '出库单'
            }
            return re
          },
          ellipsis: true
        },
        {
          title: '单据编号',
          align: 'center',
          width: 200,
          dataIndex: 'code',
          ellipsis: true
        },
        {
          title: '库存类型',
          align: 'center',
          width: 200,
          dataIndex: 'stockType',
          ellipsis: true
        },
        {
          title: '备注',
          align: 'center',
          width: 200,
          dataIndex: 'remarks',
          ellipsis: true
        },
        {
          title: '创建时间',
          align: 'center',
          width: 200,
          dataIndex: 'userName',
          ellipsis: true
        },
        {
          title: '创建人',
          align: 'center',
          width: 200,
          dataIndex: 'userId',
          ellipsis: true
        },
        {
          title: '最后更新时间',
          align: 'center',
          width: 200,
          dataIndex: 'updateTime',
          ellipsis: true
        },
        {
          title: '最小库存',
          align: 'center',
          width: 200,
          dataIndex: 'minStock',
          ellipsis: true
        },
        {
          title: '最大库存',
          align: 'center',
          width: 200,
          dataIndex: 'maxStock',
          ellipsis: true
        },
        {
          title: '安全库存',
          align: 'center',
          width: 200,
          dataIndex: 'securityStock',
          ellipsis: true
        }
      ],
      url: {
        list: '/longxianda/sysLibrary/queryStockBook',
        delete: '/longxianda/stockBook/delete',
        deleteBatch: '/longxianda/stockBook/deleteBatch',
        exportXlsUrl: '/longxianda/sysLibrary/stockBook/exportXls',
        importExcelUrl: '/longxianda/stockBook/importExcel'
      }
    }
  },
  watch: {
    $route: {
      immediate: true,
      handler() {
        // console.log('============= 哒哒哒s ============= ')
        this.queryParam.productCode = this.$route.query.productCode
        console.log('this.queryParam.productCode', this.queryParam.productCode)
        console.log('this.$route.query', this.$route.query)
        this.searchQuery()
        // console.log('$route 哒哒哒s: ',  this.queryParam.id )
      }
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
    }
  },
  created() {
    this.initColumns()
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
