<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="出库时间">
              <a-range-picker
                v-model="queryTime"
                style="width:100%"
                format="YYYY-MM-DD"
                :placeholder="['开始时间', '结束时间']"
                @change="onDateChange"
              />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="仓库">
              <a-select v-model="queryParam.storeHouse" placeholder="请选择仓库">
                <template v-for="(item, index) of storeHouseList">
                  <a-select-option :value="item.id" :key="index">{{ item.storeName }}</a-select-option>
                </template>
              </a-select>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="部门">
                <a-tree-select
                  v-model.trim="queryParam.departId"
                  style="width: 100%"
                  :tree-data="treeData"
                  placeholder="请选择"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="单号">
                <a-input v-model="queryParam.orderNum" placeholder="请输入"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="经办人">
                <a-input v-model="queryParam.operator" placeholder="请输入"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
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
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-dropdown>
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="handleExportXls('库存信息_出库主表')">导出主表</a-menu-item>
          <a-menu-item key="1" @click="handleExportDetailXls('库存明细表')">导出明细</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px" type="primary">
          <a-icon type="download"/> 导出 <a-icon type="down"
        /></a-button>
      </a-dropdown>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('库存信息表')">导出</a-button> -->
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <!-- <a-button type="primary" icon="import">导入</a-button> -->
      </a-upload>
      <!-- 高级查询区域 -->
      <!-- <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query> -->
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
    <!-- 详情表格 -->
    <div>
      <div class="title" style="margin:10px 0">
        出库明细
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        :bordered="false"
        rowKey="id"
        :columns="outList.columns"
        :dataSource="outList.dataSource"
        :pagination="outList.ipagination"
        :loading="outList.loading"
      >
      </a-table>
    </div>

    <tab-stock-modal ref="modalForm" @ok="modalFormOk"></tab-stock-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabStockModal from './modules/TabStockModal'
import { filterObj } from '@/utils/util'
import { getAction, downFile } from '@/api/manage'
import { queryDepartTreeList } from '@/api/api'

export default {
  name: 'TabStockList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabStockModal
  },
  data() {
    return {
      treeData: [],
      queryTime: [],
      storeHouseList: [],
      description: '库存信息表管理页面',
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
        {
          title: '出库单号',
          align: 'center',
          dataIndex: 'orderNum'
        },
        {
          title: '仓库',
          align: 'center',
          dataIndex: 'storeHouse'
        },
        {
          title: '库存操作类型',
          align: 'center',
          dataIndex: 'stockType'
        },
        {
          title: '部门',
          align: 'center',
          dataIndex: 'departId'
        },
        {
          title: '仓库管理员',
          align: 'center',
          dataIndex: 'stockManager'
        },
        {
          title: '经办人',
          align: 'center',
          dataIndex: 'operator'
        },
        {
          title: '客户',
          align: 'center',
          dataIndex: 'supplier'
        },
        {
          title: '数量总计',
          align: 'center',
          dataIndex: 'totalCount'
        },
        {
          title: '金额总计',
          align: 'center',
          dataIndex: 'totalPrice'
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
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      selectedRowKeys: [],
      /* table选中records*/
      selectionRows: [],
      url: {
        list: '/info/tabStock/list',
        delete: '/info/tabStock/delete',
        deleteBatch: '/info/tabStock/deleteBatch',
        exportXlsUrl: '/info/tabStock/exportXls',
        exportDetailXls: '/info/tabStock/exportDetailXls?id=',
        importExcelUrl: 'info/tabStock/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      // 出库明细
      outList: {
        loading: false,
        dataSource: [],
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
            title: '规格型号',
            align: 'center',
            dataIndex: 'spareModel'
          },
          {
            title: '计量单位',
            align: 'center',
            dataIndex: 'spareCountUnit'
          },
          // {
          //   title: '之前仓库',
          //   align: 'center',
          //   dataIndex: 'storeHouse'
          // },
          // {
          //   title: '当前仓库',
          //   align: 'center',
          //   dataIndex: 'storeHouse'
          // },
          {
            title: '数量',
            align: 'center',
            dataIndex: 'count'
          },
          {
            title: '换算单位',
            align: 'center',
            dataIndex: 'converUnit'
          },
          {
            title: '换算数量',
            align: 'center',
            dataIndex: 'converCount'
          },
          {
            title: '单价',
            align: 'center',
            dataIndex: 'unitPrice'
          },
          {
            title: '金额',
            align: 'center',
            dataIndex: 'amount'
          },

          {
            title: '备注',
            align: 'center',
            maxWidth: 500,
            ellipsis: true,
            dataIndex: 'remark'
          }
        ]
      }
    }
  },
  created() {
    this.getSuperFieldList()
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
    // 仓库
    getAction('/stock/tabStore/list').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.storeHouseList = res.result.records || []
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
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
            //update-begin---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
            this.outList.dataSource = []
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
            //update-end---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    onDateChange(value, dateString) {
      this.queryTime = value
      console.log(dateString[0], dateString[1])
      this.queryParam.startTime = dateString[0]
      this.queryParam.endTime = dateString[1]
    },
    // 导出明细
    handleExportDetailXls(fileName) {
      if (this.selectedRowKeys.length == 1) {
        console.log('this.selectedRowKeys', this.selectionRows)
        let param = this.selectionRows[0].id
        downFile(this.url.exportDetailXls + param).then(data => {
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
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            getAction('/info/tabStock/queryById?id=' + record.id).then(res => {
              if (res.success) {
                this.outList.dataSource = res.result.detailList || []
              }
            })
          },

          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    deleteHold(row) {
      console.log(row)
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      param.field = this.getQueryField()
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      param.stockInOut = 0 //0出1入
      return filterObj(param)
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'orderNum', text: '出库单号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'storeHouse', text: '仓库', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockType', text: '库存操作类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'departId', text: '部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockManager', text: '仓库管理员', dictCode: '' })
      fieldList.push({ type: 'string', value: 'operator', text: '经办人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'supplier', text: '供应商', dictCode: '' })
      // fieldList.push({ type: 'double', value: 'totalCount', text: '数量总计', dictCode: '' })
      // fieldList.push({ type: 'double', value: 'totalPrice', text: '金额总计', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
