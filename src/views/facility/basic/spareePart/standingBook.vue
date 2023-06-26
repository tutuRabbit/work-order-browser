<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-dropdown>
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="handleExportXls('备件信息_主表')">导出主表</a-menu-item>
          <a-menu-item key="2" @click="handleExportDetailXls('备件信息_明细表')">导出明细</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px" type="primary">
          <a-icon type="download"/> 导出 <a-icon type="down"
        /></a-button>
      </a-dropdown>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
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
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleTableChange"
        :customRow="handleTableClick"
      >
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
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
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
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="出入库明细" key="1">
        <standing-Book-Main-List ref="standingBookMainList"></standing-Book-Main-List>
      </a-tab-pane>
      <a-tab-pane tab="库存" key="2" forceRender>
        <inventory-List ref="inventoryList"></inventory-List>
      </a-tab-pane>
      <!-- <a-tab-pane tab="更换记录" key="3" forceRender> </a-tab-pane> -->
      <a-tab-pane tab="备件图片" key="4" forceRender>
        <Fault-pictures ref="faultPictures"></Fault-pictures>
      </a-tab-pane>
      <a-tab-pane tab="存档文档" key="5" forceRender>
        <Archived-document-List ref="archivedDocumentList"></Archived-document-List>
      </a-tab-pane>
      <!-- <a-tab-pane tab="关联设备" key="6" forceRender> </a-tab-pane> -->
    </a-tabs>
    <tab-spare-modal ref="modalForm" @ok="modalFormOk"></tab-spare-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabSpareModal from './standingBook/TabSpareModal'
import standingBookMainList from './standingBook/standingBookMainList'
import inventoryList from './standingBook/inventoryList'
import FaultPictures from './standingBook/faultPictures'
import ArchivedDocumentList from './standingBook/archivedDocumentList'
import { downFile, getAction } from '@/api/manage'

export default {
  name: 'TabSpareList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabSpareModal,
    standingBookMainList,
    ArchivedDocumentList,
    FaultPictures,
    inventoryList
  },
  data() {
    return {
      description: '备件信息管理页面',
      // 表头
      columns: [
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
        // {
        //   title: '删除标志',
        //   align: 'center',
        //   dataIndex: 'delFlag'
        // },
        {
          title: '备件编码',
          align: 'center',
          dataIndex: 'spareCode'
        },
        {
          title: '备件名称',
          align: 'center',
          dataIndex: 'spareName'
        },
        {
          title: '备件类型',
          align: 'center',
          dataIndex: 'spareType'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'spareModel'
        },
        {
          title: '生产厂商',
          align: 'center',
          dataIndex: 'spareFirm'
        },
        {
          title: '供应商',
          align: 'center',
          dataIndex: 'spareSupplier'
        },
        {
          title: '更换周期',
          align: 'center',
          dataIndex: 'spareCycle'
        },
        {
          title: '参考价',
          align: 'center',
          dataIndex: 'sparePrice'
        },
        {
          title: '计量单位',
          align: 'center',
          dataIndex: 'spareCountUnit'
        },
        {
          title: '换算单位',
          align: 'center',
          dataIndex: 'converUnit'
        },
        {
          title: '换算比例',
          align: 'center',
          dataIndex: 'converScale'
        },
        {
          title: '库存下限',
          align: 'center',
          dataIndex: 'stockLow'
        },
        {
          title: '库存上限',
          align: 'center',
          dataIndex: 'stockUp'
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
        },
        {
          title: '更换周期时间类型',
          align: 'center',
          dataIndex: 'timeType'
        },
        {
          title: '总库存',
          align: 'center',
          dataIndex: 'totalStockCount'
        },
        {
          title: '库存金额',
          align: 'center',
          dataIndex: 'totalPrice'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      loading: false,
      dataSource: [],
      spareSupplierOptions: [],
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
      url: {
        list: '/spare/tabSpare/list',
        delete: '/spare/tabSpare/delete',
        exportXlsUrl: '/spare/tabSpare/exportSpareXls', //导出主表
        exportSpareDetailXls: '/spare/tabSpare/exportSpareDetailXls', //导出明细
        importExcelUrl: 'spare/tabSpare/importExcel',
        deleteBatch: '/spare/tabSpare/deleteBatch'
      },
      dictOptions: {},

      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
    // 生产商、供应商
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      // getAction('/equipment/tabContactCompany/list').then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        arr.forEach(item => {
          let pm = {
            value: item.id,
            label: item.companyName
          }
          this.spareSupplierOptions.push(pm)
        })
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.standingBookMainList.details(record) //出入库明细
            this.$refs.inventoryList.details(record) //库存
            this.$refs.faultPictures.details(record) //备件图片
            this.$refs.archivedDocumentList.details(record) //存档文档
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    // 导出明细
    handleExportDetailXls(fileName) {
      if (this.selectedRowKeys.length == 1) {
        console.log('this.selectedRowKeys', this.selectionRows)
        let param = this.selectionRows[0].id
        downFile(this.url.exportSpareDetailXls + '?id=' + param).then(data => {
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
            link.setAttribute('download', fileName + '.xls')
            document.body.appendChild(link)
            link.click()
            document.body.removeChild(link) //下载完成移除元素
            window.URL.revokeObjectURL(url) //释放掉blob对象
          }
        })
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'spareCode', text: '备件编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'spareName', text: '备件名称', dictCode: '' })
      fieldList.push({ type: 'select', value: 'spareType', text: '备件类型', dictCode: 'spare_type' })
      fieldList.push({ type: 'string', value: 'spareModel', text: '规格型号', dictCode: '' })
      fieldList.push({ type: 'select', value: 'spareFirm', text: '生产厂商', options: this.spareSupplierOptions })
      fieldList.push({ type: 'select', value: 'spareSupplier', text: '供应商', options: this.spareSupplierOptions })
      // fieldList.push({ type: 'int', value: 'spareCycle', text: '更换周期', dictCode: '' })
      fieldList.push({ type: 'number', value: 'sparePrice', text: '参考价', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'spareCountUnit', text: '计量单位', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'converUnit', text: '换算单位', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'converScale', text: '换算比例', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'stockLow', text: '库存下限', dictCode: '' })
      // fieldList.push({ type: 'int', value: 'stockUp', text: '库存上限', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'timeType', text: '更换周期时间类型', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
