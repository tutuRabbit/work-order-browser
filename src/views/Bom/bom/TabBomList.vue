<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="BOM单组别">
              <a-input v-model="queryParam.bomGroup" placeholder="请输入"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="BOM单编号">
              <a-input v-model="queryParam.bomCode" placeholder="请输入"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="版本">
                <a-input v-model="queryParam.version" placeholder="请输入"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="状态">
                <a-radio-group v-model="queryParam.status">
                  <a-radio value="0">使用</a-radio>
                  <a-radio value="1">未使用</a-radio>
                </a-radio-group>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="图号">
                <a-input v-model="queryParam.pictureNum" placeholder="请输入"></a-input>
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
      <a-button @click="handleAdd" v-has="'bom:add'" type="primary" icon="plus">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('产品物料BOM单')">导出</a-button> -->
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'bom:delete'">
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
        bordered
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
          <a @click="handleEdit(record)" v-has="'bom:edit'">编辑</a>
          <a-divider type="vertical" />
          <a @click="handleDetail(record)" v-has="'bom:view'">详情</a>

          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" v-has="'bom:delete'" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)" v-has="'bom:view'">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" v-has="'bom:delete'" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <!-- 详情表格 -->
    <div>
      <div class="title" style="margin:10px 0">
        物料明细
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="outList.columns"
        :dataSource="outList.dataSource"
        :pagination="outList.ipagination"
        :loading="outList.loading"
      >
      </a-table>
    </div>

    <tab-bom-modal ref="modalForm" @ok="modalFormOk"></tab-bom-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabBomModal from './modules/TabBomModal'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { filterObj } from '@/utils/util'
import { getAction, downFile } from '@/api/manage'
import { queryDepartTreeList } from '@/api/api'

export default {
  name: 'TabBomList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabBomModal
  },
  data() {
    return {
      treeData: [],
      queryTime: [],
      description: '产品物料BOM单管理页面',
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
          title: 'BOM单组别',
          align: 'center',
          dataIndex: 'bomGroup'
        },
        {
          title: 'BOM单编号',
          align: 'center',
          dataIndex: 'bomCode'
        },
        {
          title: '版本',
          align: 'center',
          dataIndex: 'version'
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'status'
        },
        {
          title: '图号',
          align: 'center',
          dataIndex: 'pictureNum'
        },
        {
          title: '产品编码',
          align: 'center',
          dataIndex: 'productCode'
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '规格',
          align: 'center',
          dataIndex: 'productNorms'
        },
        {
          title: '产品属性',
          align: 'center',
          dataIndex: 'productProperty'
        },
        {
          title: '单位',
          align: 'center',
          dataIndex: 'stockPiece'
        },
        {
          title: '工艺路线',
          align: 'center',
          dataIndex: 'craftRouteName'
        },
        {
          title: '数量',
          align: 'center',
          dataIndex: 'stockNumber'
        },
        {
          title: '成品率',
          align: 'center'
          // dataIndex: 'pictureNum'
        },
        {
          title: '建立人员',
          align: 'center',
          dataIndex: 'createBy'
        },
        {
          title: '建立日期',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '最后更新人员',
          align: 'center',
          dataIndex: 'updateBy'
        },
        {
          title: '最后更新日期',
          align: 'center',
          dataIndex: 'updateTime'
        },

        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 200,
          scopedSlots: { customRender: 'action' }
        }
      ],
      dictOptions: {},
      selectedRowKeys: [],
      /* table选中records*/
      selectionRows: [],
      url: {
        list: '/bom/tabBom/list',
        delete: '/bom/tabBom/delete',
        deleteBatch: '/bom/tabBom/deleteBatch',
        exportXlsUrl: '/bom/tabBom/exportXls',
        importExcelUrl: '/bom/tabBom/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      // 物料明细
      outList: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '顺序号',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function(t, r, index) {
              return parseInt(index) + 1
            }
          },
          {
            title: '物料代码',
            align: 'center',
            dataIndex: 'lastProductCode'
          },
          {
            title: '物料名称',
            align: 'center',
            dataIndex: 'lastProductName'
          },
          {
            title: '物料属性',
            align: 'center'
            // dataIndex: '11'
          },
          {
            title: '单位',
            align: 'center',
            dataIndex: 'lastProductUnitName'
          },
          {
            title: '基本单位',
            align: 'center'
            // dataIndex: 'lastProductUnitName'
          },
          {
            title: '子项类型',
            align: 'center'
            // dataIndex: 'converUnit'
          },
          {
            title: '基本单位用量',
            align: 'center',
            dataIndex: 'unitQty'
          },
          {
            title: '用量',
            align: 'center'
            // dataIndex: 'unitPrice'
          },
          {
            title: '损耗率(%)',
            align: 'center'
            // dataIndex: 'amount'
          },
          {
            title: '位置号',
            align: 'center'
            // dataIndex: 'amount'
          },
          {
            title: '胚料尺寸',
            align: 'center'
            // dataIndex: 'amount'
          },
          {
            title: '胚料数',
            align: 'center'
            // dataIndex: 'amount'
          },
          {
            title: '工序号',
            align: 'center'
            // dataIndex: 'amount'
          }
        ]
      }
    }
  },
  created() {
    this.getSuperFieldList()
    // 库存操作类型
    // getAction('/Bom/tabStore/list').then(res => {
    //   if (res.success) {
    //     this.$set(this.dictOptions, 'BomType', res.result.records)
    //     console.log('res.result.records', res.result.records)
    //   }
    // })
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
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
            getAction('/bom/tabBom/queryById?id=' + record.id).then(res => {
              if (res.success) {
                this.outList.dataSource = res.result.materials || []
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
      return filterObj(param)
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'orderNum', text: '入库单号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'storeHouse', text: '仓库', dictCode: '' })
      fieldList.push({ type: 'string', value: 'BomType', text: '库存操作类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'departId', text: '部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'BomManager', text: '仓库管理员', dictCode: '' })
      fieldList.push({ type: 'string', value: 'operator', text: '经办人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'supplier', text: '供应商', dictCode: '' })
      fieldList.push({ type: 'double', value: 'totalCount', text: '数量总计', dictCode: '' })
      fieldList.push({ type: 'double', value: 'totalPrice', text: '金额总计', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
