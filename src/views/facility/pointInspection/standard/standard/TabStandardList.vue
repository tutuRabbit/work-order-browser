<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="设备类型">
              <j-category-select v-model="queryParam.equipmentType" pcode="B03" placeholder="请选择设备类型" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="巡检目的">
              <a-input v-model.trim="queryParam.patrolInspection" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="快速查询">
              <a-input v-model.trim="queryParam.standardName" placeholder="标准名称"> </a-input>
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
        :customRow="handleTableClick"
        @change="handleTableChange"
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
    <a-tabs defaultActiveKey="0">
      <a-tab-pane tab="巡检项明细" key="0" forceRender>
        <Operating-record-List ref="operatingRecordList"></Operating-record-List>
      </a-tab-pane>
    </a-tabs>
    <tab-standard-modal ref="modalForm" @ok="modalFormOk"></tab-standard-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { deleteAction, getAction, downFile, getFileAccessHttpUrl } from '@/api/manage'
import { filterObj } from '@/utils/util'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabStandardModal from './modules/TabStandardModal'
import OperatingRecordList from './modules/operatingRecordList'

export default {
  name: 'TabStandardList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    OperatingRecordList,
    TabStandardModal
  },
  data() {
    return {
      description: '点巡检标准表管理页面',
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
          title: '标准名称',
          align: 'center',
          width: '20%',
          ellipsis: true,
          resizable: true,
          dataIndex: 'standardName'
        },
        {
          title: '设备类型',
          align: 'center',
          width: '20%',
          ellipsis: true,
          resizable: true,
          dataIndex: 'equipmentType'
        },
        {
          title: '巡检目的',
          align: 'center',
          width: '20%',
          ellipsis: true,
          resizable: true,
          dataIndex: 'patrolInspection'
        },
        {
          title: '备注',
          align: 'center',
          dataIndex: 'remark',
          width: '20%',
          ellipsis: true,
          resizable: true
        },
        {
          title: '检查项总数',
          align: 'center',
          dataIndex: 'checkTotal',
          width: '20%',
          ellipsis: true,
          resizable: true
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
        list: '/checkout/pointpatrolstandard/page',
        delete: '/checkout/pointpatrolstandard/delete',
        deleteBatch: '/checkout/pointpatrolstandard/deleteBatch',
        exportXlsUrl: '/checkout/pointpatrolstandard/exportXls',
        importExcelUrl: 'checkout/pointpatrolstandard/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      queryTime: [],
      isorterI: {
        column: 'createTime',
        orderBy: 'desc'
      }
    }
  },
  created() {},
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handleDelete: function(id) {
      if (!this.url.delete) {
        this.$message.error('请设置url.delete属性!')
        return
      }
      var that = this
      deleteAction(that.url.delete, { id: id }).then(res => {
        if (res.code == 200) {
          //重新计算分页问题
          that.reCalculatePage(1)
          that.$message.success(res.msg)
          that.loadData()
        } else {
          that.$message.warning(res.msg)
        }
      })
    },
    batchDel: function() {
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
          title: '确认删除',
          content: '是否删除选中数据?',
          onOk: function() {
            that.loading = true
            deleteAction(that.url.deleteBatch, { ids: ids })
              .then(res => {
                if (res.code == 200) {
                  //重新计算分页问题
                  that.reCalculatePage(that.selectedRowKeys.length)
                  that.$message.success(res.msg)
                  that.loadData()
                  that.onClearSelected()
                } else {
                  that.$message.warning(res.msg)
                }
              })
              .finally(() => {
                that.loading = false
              })
          }
        })
      }
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorterI, this.filters)
      param.status = '0'
      param.pageNum = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
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
          if (res.code == 200) {
            this.dataSource = res.data.records || res.data
            if (res.total) {
              this.ipagination.total = res.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.msg)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.operatingRecordList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
</style>
