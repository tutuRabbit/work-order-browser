<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="使用部门">
              <a-tree-select
                v-model.trim="queryParam.sysOrgCode"
                style="width: 100%"
                :tree-data="treeData"
                :field-names="{ label: 'title', value: 'value', children: 'children' }"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                placeholder="请选择使用部门"
              >
              </a-tree-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="12">
            <a-form-item label="快速查询">
              <a-input v-model="queryParam.search" placeholder="设备名称、编码、规格型号"></a-input>
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
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('设备表')">导出</a-button>
      <!-- <a-dropdown>
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="handleExportXls('设备台账_主表')">导出主表</a-menu-item>
          <a-menu-item key="1" @click="handleExportDetailXls('设备台账_主表+图片')">导出明细</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px" type="primary">
          <a-icon type="download"/> 导出 <a-icon type="down"
        /></a-button>
      </a-dropdown> -->
      <a-button type="primary" icon="download" @click="handleExportDetailXls('设备档案表')">导出设备档案</a-button>
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
      <a-dropdown>
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="addSubset">添加子设备</a-menu-item>
          <a-menu-item key="2" @click="delSubset">移除子设备</a-menu-item>
        </a-menu>
        <a-button type="primary" style="margin-left: 8px"
          ><a-icon type="share-alt"/> 子设备 <a-icon type="down"
        /></a-button>
      </a-dropdown>
      <a-dropdown>
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="listShow(0)">所有设备按列表展示</a-menu-item>
          <a-menu-item key="2" @click="listShow(1)">主子设备按树形展示</a-menu-item>
          <!-- <a-menu-item key="3">列隐藏展示</a-menu-item> -->
        </a-menu>
        <a-button type="primary" style="margin-left: 8px"
          ><a-icon type="pic-right"/> 设备布局 <a-icon type="down"
        /></a-button>
      </a-dropdown>
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
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :data-source="dataSource"
        :pagination="ipagination"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        :loading="loading"
        @change="handleTableChange"
        class="j-table-force-nowrap"
        :scroll="{ y: 200 }"
        :customRow="handleTableClick"
        childrenColumnName="subEquipmentList"
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
      <a-tab-pane tab="故障报修" key="1">
        <Malfunction-repair ref="malfunctionRepair"></Malfunction-repair>
      </a-tab-pane>
      <a-tab-pane tab="维修计划" key="2" forceRender> </a-tab-pane>
    </a-tabs>
    <equipment-modal ref="modalForm" @ok="modalFormOk"></equipment-modal>
    <!-- 添加子设备 -->
    <j-modal
      title="添加子设备"
      width="75%"
      :visible="SubsetVis"
      @ok="handleOk"
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div
        style="
                background-color: #f4f4f4;
                font-size: 18px;
                color: #397cc0;
                font-weight: 600;
                width: 100%;
                margin: -5px 0 15px 0;"
      >
        <span style="margin: 0 10px">设备编码:</span>
        <span style="margin: 0 10px">{{ this.SubsetEquipmentCode }}</span>
        <span style="margin: 0 10px">设备名称:</span>
        <span style="margin: 0 10px">{{ this.SubsetEquipmentName }}</span>
      </div>
      <!-- 操作按钮区域 -->
      <div class="table-operator">
        <a-button @click="handleSubset" type="primary" icon="plus">选择设备</a-button>
        <!-- <a-button type="primary" icon="delete">删除选中</a-button> -->
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="SubsetData.columns"
        :dataSource="SubsetData.dataSource"
        :pagination="SubsetData.pagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        childrenColumnName="subEquipmentList"
      >
        <template v-slot:action="props">
          <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
            <a>删除</a>
          </a-popconfirm>
        </template>
      </a-table>
    </j-modal>
    <!-- 子设备表格 -->
    <j-modal
      title="选择设备"
      width="55%"
      :visible="SubsetTabVis"
      @ok="SubsetTabHandleOk"
      @cancel="SubsetTabVis = false"
      cancelText="关闭"
    >
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="SubsetTabQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="设备编码，名称，型号" v-model="SubsetTabParam.search"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="设备类型">
                <j-category-select v-model="SubsetTabParam.equipmentType" pcode="B03" placeholder="请选择设备类型" />
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model="SubsetTabParam.sysOrgCode"
                  style="width: 100%"
                  :tree-data="treeData"
                  placeholder="请选择使用部门"
                />
              </a-form-item>
            </a-col>
            <a-col :span="22">
              <span style="float: right;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="SubsetTabQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="SubsetTabReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="SubsetTab"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="SubsetTab.columns"
        :dataSource="SubsetTab.dataSource"
        :pagination="SubsetTab.pagination"
        :loading="SubsetTab.loading"
        @change="SubHandleTableChange"
        :rowSelection="{ selectedRowKeys: SubsetTabRowKeys, onChange: SubsetTabChange }"
        class="j-table-force-nowrap"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { mixinDevice } from '@/utils/mixin'
import { loadCategoryData } from '@/api/api'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import EquipmentModal from './EquipmentModal'
import MalfunctionRepair from './malfunctionRepair'
import { getAction, postAction, deleteAction, downFile } from '@/api/manage'
import { filterObj } from '@/utils/util'

export default {
  name: 'EquipmentList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    EquipmentModal,
    MalfunctionRepair
  },
  props: ['value'],
  data() {
    return {
      description: '设备表管理页面',
      //   添加子设备
      SubsetTabRowKeys: [],
      SubsetTabRows: [],
      treeData: [],
      SubsetTabParam: {},
      SubsetVis: false,
      SubsetTabVis: false,
      disableSubmit: false,
      SubsetEquipmentCode: '',
      SubsetEquipmentName: '',
      SubsetId: null,
      SubsetData: {
        loading: false,
        dataSource: [],
        pagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        columns: [
          {
            title: '设备编码',
            align: 'center',
            dataIndex: 'equipmentCode'
          },
          {
            title: '设备名称',
            align: 'center',
            dataIndex: 'equipmentName'
          },
          {
            title: '规格型号',
            align: 'center',
            dataIndex: 'equipmentModel'
          },
          {
            title: '使用部门',
            align: 'center',
            dataIndex: 'depart'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      SubsetTab: {
        loading: false,
        dataSource: [],
        pagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        columns: [
          {
            title: '设备编码',
            align: 'center',
            dataIndex: 'equipmentCode'
          },
          {
            title: '设备名称',
            align: 'center',
            dataIndex: 'equipmentName'
          },
          {
            title: '规格型号',
            align: 'center',
            dataIndex: 'equipmentModel'
          },
          {
            title: '使用部门',
            align: 'center',
            dataIndex: 'depart'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      listShows: 0,
      expandedRowKeys: [],
      //   end
      // 表头
      columns: [
        // {
        //   title: '#',
        //   dataIndex: '',
        //   key: 'rowIndex',
        //   width: 60,
        //   align: 'center',
        //   customRender: function(t, r, index) {
        //     return parseInt(index) + 1
        //   }
        // },
        {
          title: '设备编码',
          align: 'center',
          dataIndex: 'equipmentCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'equipmentName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'equipmentModel'
        },
        {
          title: '设备类型',
          align: 'center',
          dataIndex: 'equipmentType',
          customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
        },
        {
          title: '部门名称',
          align: 'center',
          dataIndex: 'depart'
        },
        {
          title: '操作人',
          align: 'center',
          dataIndex: 'equipmentOperator'
        },
        {
          title: '生产厂商',
          align: 'center',
          dataIndex: 'equipmentFirm'
        },
        {
          title: '供应商',
          align: 'center',
          dataIndex: 'equipmentSupplier'
        },
        // {
        //   title: '使用寿命',
        //   align: 'center',
        //   dataIndex: 'equipmentAge'
        // },
        {
          title: '设备位置',
          align: 'center',
          dataIndex: 'equipmentAddress'
        },
        {
          title: '使用状况',
          align: 'center',
          dataIndex: 'equipmentStatus'
        },
        {
          title: '购置时间',
          align: 'center',
          dataIndex: 'equipmentBuyTime'
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
      url: {
        list: '/equipment/equipment/list',
        delete: '/equipment/equipment/delete',
        deleteBatch: '/equipment/equipment/deleteBatch',
        exportXlsUrl: '/equipment/equipment/exportXls',
        importEquipmentExcel: '/equipment/equipment/importEquipmentExcel', //导出设备档案
        notSubList: '/equipment/equipment/notSubList', //添加子设备-选择设备
        importExcelUrl: 'equipment/equipment/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      spareSupplierOptions: []
    }
  },
  watch: {
    value: {
      immediate: true,
      handler(equipmentType) {
        console.log('equipmentType', equipmentType)
        this.dataSource = []
        this.loadData(1, equipmentType)
      }
    }
  },
  created() {
    this.getSuperFieldList() // 这个方法是找到所有的部门信息
    //生产商、供应商
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
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
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.malfunctionRepair.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    // 设备档案表
    handleExportDetailXls(fileName) {
      if (this.selectedRowKeys.length == 1) {
        console.log('this.selectedRowKeys', this.selectionRows)
        let param = this.selectionRows[0].id
        downFile(this.url.importEquipmentExcel + '?equipmentId=' + param).then(data => {
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
    handleExpand(expanded, record) {
      this.expandedRowKeys = []
      this.SubsetTab.dataSource = []
      if (expanded === true) {
        this.loading = true
        this.expandedRowKeys.push(record.id)
        getAction(this.url.list).then(res => {
          if (res.success) {
            this.loading = false
            this.SubsetTab.dataSource = res.result.records
          }
        })
      }
    },
    loadData(pageNum, equipmentType) {
      this.loading = true
      if (pageNum === 1) {
        this.ipagination.current = 1
      }
      if (!equipmentType) {
        getAction(this.url.list, {
          ...this.getQueryParams()
        })
          .then(res => {
            if (res.success) {
              this.dataSource = res.result.records
              // this.dataSource = this.getDataByResult(res.result.records)
              console.log(' this.dataSource ', this.dataSource)
              this.ipagination.total = res.result.total
            }
          })
          .finally(() => {
            this.loading = false
            this.cardLoading = false
          })
      } else {
        //加载数据 若传入参数1则加载第一页的内容
        getAction(this.url.list, {
          equipmentType,
          ...this.getQueryParams()
        })
          .then(res => {
            if (res.success) {
              this.dataSource = res.result.records
              this.ipagination.total = res.result.total
            }
          })
          .finally(() => {
            this.loading = false
            this.cardLoading = false
          })
      }
    },
    getDataByResult(result) {
      console.log('result', result)
      if (result && result.length > 0) {
        return result.map(item => {
          //判断是否标记了带有子节点
          if (item[this.hasChildrenField] == '1') {
            let loadChild = { id: item.id + '_loadChild', name: 'loading...', isLoading: true }
            console.log('loadChild', loadChild)
            item.children = [loadChild]
            console.log('item.children', item.children)
          }
          console.log('item', item)
          return item
        })
      }
    },
    searchQuery() {
      this.loadData(1, this.value)
    },
    // 选择设备
    listShow(val) {
      console.log('val', val)
      this.listShows = val
      this.loadData(1, this.value)
    },
    close() {
      this.SubsetVis = false
      this.SubsetEquipmentCode = ''
      this.SubsetEquipmentName = ''
      this.SubsetData.dataSource = []
      //   this.SubsetTabVis = false
      //   this.getAllTable().then(editableTables => {
      //     this.table.dataSource = []
      //   })
      this.$emit('close')
    },
    handleCancel() {
      this.close()
    },
    handleOk() {
      let that = this
      let temp = that.SubsetData.dataSource.map(item => item.id)
      var ids = ''
      for (var a = 0; a < temp.length; a++) {
        ids += temp[a] + ','
      }
      ids = ids.slice(0, -1)
      let parm = {
        equipmentId: that.SubsetId,
        subEquipementIds: ids
      }
      postAction('/SubEquipementLink/subEquipementLink/add', parm)
        .then(res => {
          console.log('res', res)
          if (res.success) {
            that.$message.success(res.message)
            this.loadData()
            // that.$emit('ok')
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(err => {
          that.SubsetVis = false
        })
    },
    SubsetTabHandleOk() {
      this.SubsetData.dataSource = this.SubsetTabRows
      this.SubsetTabVis = false
    },
    addSubset() {
      if (this.selectedRowKeys.length == 1) {
        var ids = ''
        for (var a = 0; a < this.selectionRows.length; a++) {
          ids = this.selectionRows[a]
        }
        this.SubsetEquipmentCode = ids.equipmentCode
        this.SubsetEquipmentName = ids.equipmentName
        this.SubsetId = ids.id
        this.SubsetVis = true
      } else {
        this.$message.warning('请选择一条记录！')
        return
      }
    },
    delSubset() {
      if (!this.url.deleteBatch) {
        this.$message.error('请设置url.deleteBatch属性!')
        return
      }
      if (this.selectedRowKeys.length <= 0) {
        this.$message.warning('请选择一条记录！')
        return
      } else {
        var ids = ''
        for (var a = 0; a < this.selectedRowKeys.length; a++) {
          ids += this.selectedRowKeys[a] + ','
        }
        var that = this
        this.$confirm({
          title: '移除子设备',
          content: '确定移除该子设备吗?',
          onOk: function() {
            that.loading = true
            deleteAction('/SubEquipementLink/subEquipementLink/deleteBatch', { ids: ids })
              .then(res => {
                if (res.success) {
                  //重新计算分页问题
                  that.reCalculatePage(that.selectedRowKeys.length)
                  that.$message.success(res.message)
                  that.loadData()
                  that.onClearSelected()
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                that.loading = false
              })
          }
        })
      }
    },
    handleSubset() {
      this.SubsetTabVis = true
      this.SubsetTabLoadData()
    },
    SubsetTabChange(SubsetTabRowKeys, SubsetTabRows) {
      this.SubsetTabRowKeys = SubsetTabRowKeys
      this.SubsetTabRows = SubsetTabRows
    },
    SubHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      console.log(pagination)
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.SubsetTab.pagination = pagination
      this.SubsetTabLoadData()
    },
    SubsetTabQuery() {
      this.SubsetTabLoadData(1)
    },
    SubsetTabReset() {
      this.SubsetTabParam = {}
      this.SubsetTabLoadData(1)
    },
    SubsetTabLoadData(arg) {
      if (!this.url.notSubList) {
        this.$message.error('请设置url.notSubList属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.SubsetTab.pagination.current = 1
      }
      var params = {
        id: this.SubsetId,
        pageNo: this.SubsetTab.pagination.current,
        pageSize: this.SubsetTab.pagination.pageSize
      }
      this.SubsetTab.loading = true
      getAction(this.url.notSubList, params)
        .then(res => {
          console.log('res.result', res.result)
          if (res.success) {
            this.SubsetTab.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.SubsetTab.pagination.total = res.result.total
            } else {
              this.SubsetTab.pagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.SubsetTab.loading = false
        })
    },
    // end
    getQueryParams(val) {
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
      if (!val) {
        param.model = this.listShows
      }
      return filterObj(param)
    },
    searchReset() {
      this.queryParam = {}
      this.loadData(1, this.value)
    },
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          console.log('res.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'equipmentCode', text: '设备编码' })
      fieldList.push({ type: 'string', value: 'equipmentName', text: '设备名称' })
      fieldList.push({ type: 'string', value: 'equipmentAddress', text: '设备位置' })
      fieldList.push({ type: 'string', value: 'equipmentModel', text: '规格型号' })
      fieldList.push({ type: 'select', value: 'equipmentFirm', text: '生产厂商', options: this.spareSupplierOptions })
      fieldList.push({ type: 'select', value: 'equipmentSupplier', text: '供应商', options: this.spareSupplierOptions })
      fieldList.push({ type: 'string', value: 'equipmentManager', text: '资产负责人' })
      fieldList.push({ type: 'select', value: 'equipmentStatus', text: '使用状况', dictCode: 'equipment_use_status' })
      fieldList.push({
        type: 'select',
        value: 'equipmentMarks',
        text: '设备标记',
        options: [
          { label: '重点设备', value: '重点设备' },
          { label: '主要设备', value: '主要设备' },
          { label: '一般设备', value: '一般设备' }
        ]
      })
      fieldList.push({ type: 'string', value: 'equipmentOperator', text: '操作人', dictCode: 'sys_user,realname,id' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
::v-deep {
  .ant-tabs-bar {
    margin: 0;
  }
}
</style>
