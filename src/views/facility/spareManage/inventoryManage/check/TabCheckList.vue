<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="开始时间">
              <j-date
                style="width:100%"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                v-model="queryParam.startTime"
              ></j-date>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="结束时间">
              <j-date
                style="width:100%"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                v-model="queryParam.endTime"
              ></j-date>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="仓库">
                <!-- <a-input v-model="queryParam.delFlag"></a-input> -->
                <a-select v-model="queryParam.storeHouse">
                  <template v-for="(item, index) of storeHouseList">
                    <a-select-option :value="item.id" :key="index">{{ item.storeName }}</a-select-option>
                  </template>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="盘点编号">
                <a-input v-model="queryParam.checkCode"></a-input>
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
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">添加盘点单</a-button>
      <!-- <a-button type="primary" icon="import" @click="handleExportXls('盘点单表')">导出</a-button> -->
      <a-button type="primary" icon="import" @click="handleExportDetailXls('盘点单表')">导出</a-button>

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
      <a-button type="primary" icon="print">打印</a-button>
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
        盘点明细
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
    <tab-check-modal ref="modalForm" @ok="modalFormOk"></tab-check-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { getAction } from '@/api/manage'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabCheckModal from './modules/TabCheckModal'
import { downFile } from '@/api/manage'

export default {
  name: 'TabCheckList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabCheckModal
  },
  data() {
    return {
      // 盘点明细
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
          // {
          //   title: '计量单位',
          //   align: 'center',
          //   dataIndex: 'spareCountUnit'
          // },
          {
            title: '账面数量',
            align: 'center',
            dataIndex: 'totalStockCount'
          },
          {
            title: '实际数量',
            align: 'center',
            dataIndex: 'realCount'
          },
          {
            title: '盘盈数量',
            align: 'center',
            dataIndex: 'profitCount'
          },
          {
            title: '盘亏数量',
            align: 'center',
            dataIndex: 'lossCount'
          },
          // {
          //   title: '换算单位',
          //   align: 'center',
          //   dataIndex: 'departId_dictText'
          // },
          // {
          //   title: '原位置编码',
          //   align: 'center',
          //   dataIndex: 'stockManager'
          // },
          // {
          //   title: '当前位置编码',
          //   align: 'center',
          //   dataIndex: 'stockManager'
          // },
          {
            title: '备注',
            align: 'center',
            dataIndex: 'remark'
          }
        ]
      },
      description: '盘点单表管理页面',
      storeHouseList: [],
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
          title: '盘点编号',
          align: 'center',
          dataIndex: 'checkCode'
        },
        {
          title: '仓库',
          align: 'center',
          dataIndex: 'storeHouse'
        },
        {
          title: '经办人',
          align: 'center',
          dataIndex: 'operator'
        },
        {
          title: '备注',
          align: 'center',
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
      url: {
        list: '/check/tabCheck/list',
        delete: '/check/tabCheck/delete',
        deleteBatch: '/check/tabCheck/deleteBatch',
        exportXlsUrl: '/check/tabCheck/exportXls',
        importExcelUrl: 'check/tabCheck/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
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
    // 导出
    handleExportDetailXls(fileName) {
      if (this.selectedRowKeys.length == 1) {
        let param = this.selectionRows[0].id
        downFile(this.url.exportXlsUrl + '?id=' + param).then(data => {
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
            let timesTemp = Date.parse(new Date())
            console.log('timesTemp', timesTemp)
            timesTemp = timesTemp / 1000
            link.setAttribute('download', fileName + '-' + timesTemp + '.xls')
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
            getAction('/check/tabCheck/queryTabCheckDetails?id=' + record.id).then(res => {
              if (res.success) {
                this.outList.dataSource = res.result.records || []
              }
            })
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'datetime', value: 'updateTime', text: '更新日期' })
      fieldList.push({ type: 'string', value: 'sysOrgCode', text: '所属部门', dictCode: '' })
      fieldList.push({ type: 'string', value: 'checkCode', text: '盘点编号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'storeHouse', text: '仓库', dictCode: '' })
      fieldList.push({ type: 'string', value: 'operator', text: '经办人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
