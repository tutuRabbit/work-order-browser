<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="设备编号">
              <a-input v-model.trim="queryParam.deviceCode" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备名称">
              <a-input v-model.trim="queryParam.deviceName" placeholder="请输入"> </a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="点检时间">
                <a-range-picker
                  :value="queryTime"
                  style="width:100%"
                  :placeholder="['开始时间', '结束时间']"
                  @change="onChangeRangeDate"
                  format="YYYY-MM-DD"
                />
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="点检人">
                <!-- <a-input v-model.trim="queryParam.spotCheckPerson" placeholder="请输入"> </a-input> -->
                <j-dict-select-tag
                  v-model="queryParam.spotCheckPerson"
                  placeholder="请选择点检人"
                  dictCode="sys_user,realname,id"
                />
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
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
      <a-button type="primary" icon="download" @click="handleExportXls('点检记录表')">导出</a-button>
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
    <div class="title">点检明细</div>
    <Operating-record-List ref="operatingRecordList"></Operating-record-List>
    <tab-spot-check-records-modal ref="modalForm" @ok="modalFormOk"></tab-spot-check-records-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import OperatingRecordList from './modules/operatingRecordList'
import TabSpotCheckRecordsModal from './modules/TabSpotCheckRecordsModal'

export default {
  name: 'TabSpotCheckRecordsList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    OperatingRecordList,
    TabSpotCheckRecordsModal
  },
  data() {
    return {
      description: '点检记录表管理页面',
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
        //   title: '点检项id',
        //   align: 'center',
        //   dataIndex: 'spotCheckItemId'
        // },
        {
          title: '设备编号',
          align: 'center',
          dataIndex: 'deviceCode'
        },
        {
          title: '设备名称',
          align: 'center',
          dataIndex: 'deviceName'
        },
        {
          title: '点检时间',
          align: 'center',
          dataIndex: 'spotCheckTime'
        },
        {
          title: '点检人',
          align: 'center',
          dataIndex: 'spotCheckPerson'
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
        list: '/spot_record/tabSpotCheckRecords/list',
        delete: '/spot_record/tabSpotCheckRecords/delete',
        deleteBatch: '/spot_record/tabSpotCheckRecords/deleteBatch',
        exportXlsUrl: '/spot_record/tabSpotCheckRecords/exportXls',
        importExcelUrl: 'spot_record/tabSpotCheckRecords/importExcel'
      },
      dictOptions: {},
      superFieldList: [],
      queryTime: []
    }
  },
  created() {
    this.getSuperFieldList()
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
            this.$refs.operatingRecordList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    initDictConfig() {},
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.lastSpotCheckStartTime = dateString[0]
      this.queryParam.lastSpotCheckEndTime = dateString[1]
      // console.log('this.queryParam.lastSpotCheckStartTime', this.queryParam)
    },
    getSuperFieldList() {
      // let fieldList = []
      // fieldList.push({ type: 'string', value: 'spotCheckItemId', text: '点检项id', dictCode: '' })
      // fieldList.push({ type: 'datetime', value: 'spotCheckTime', text: '点检时间' })
      // fieldList.push({ type: 'string', value: 'spotCheckPerson', text: '点检人', dictCode: '' })
      // fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      // this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
