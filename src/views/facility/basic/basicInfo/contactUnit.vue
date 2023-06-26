<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="快速查询">
              <a-input v-model="queryParam.search" placeholder="输单位名称、单位编码、联系人"></a-input>
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
      <a-button @click="handleAdd" v-has="'permission:add'" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" v-has="'permission:export'" icon="download" @click="handleExportXls('往来单位')"
        >导出</a-button
      >
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button v-has="'permission:import'" type="primary" icon="import">导入</a-button>
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
          <a @click="handleEdit(record)" v-has="'permission:edit'">编辑</a>
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

    <tab-contact-company-modal ref="modalForm" @ok="modalFormOk"></tab-contact-company-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabContactCompanyModal from './contactUnit/TabContactCompanyModal'

export default {
  name: 'contactUnit',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TabContactCompanyModal
  },
  data() {
    return {
      description: '往来单位管理页面',
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
          title: '单位编码',
          align: 'center',
          dataIndex: 'companyCode'
        },
        {
          title: '单位名称',
          align: 'center',
          dataIndex: 'companyName'
        },
        {
          title: '默认联系人',
          align: 'center',
          dataIndex: 'defaultContact'
        },
        {
          title: 'qq',
          align: 'center',
          dataIndex: 'qq'
        },
        {
          title: '微信',
          align: 'center',
          dataIndex: 'wechat'
        },
        {
          title: '电话',
          align: 'center',
          dataIndex: 'tel'
        },
        {
          title: '邮箱',
          align: 'center',
          dataIndex: 'email'
        },
        {
          title: '传真',
          align: 'center',
          dataIndex: 'fax'
        },
        {
          title: '税务登记号',
          align: 'center',
          dataIndex: 'taxRegistration'
        },
        {
          title: '地址',
          align: 'center',
          dataIndex: 'address'
        },
        {
          title: '邮编',
          align: 'center',
          dataIndex: 'zipCode'
        },
        {
          title: '开户行',
          align: 'center',
          dataIndex: 'openingBank'
        },
        {
          title: '银行帐户',
          align: 'center',
          dataIndex: 'bankAccount'
        },
        {
          title: '网址',
          align: 'center',
          dataIndex: 'website'
        },
        {
          title: '备注',
          align: 'center',
          dataIndex: 'note'
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
        list: '/equipment/tabContactCompany/list',
        delete: '/equipment/tabContactCompany/delete',
        deleteBatch: '/equipment/tabContactCompany/deleteBatch',
        exportXlsUrl: '/equipment/tabContactCompany/exportXls',
        importExcelUrl: 'equipment/tabContactCompany/importExcel'
      },
      dictOptions: {},
      superFieldList: []
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
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'companyCode', text: '单位编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'companyName', text: '单位名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'defaultContact', text: '默认联系人', dictCode: '' })
      fieldList.push({ type: 'string', value: 'qq', text: 'qq', dictCode: '' })
      fieldList.push({ type: 'string', value: 'wechat', text: '微信', dictCode: '' })
      fieldList.push({ type: 'string', value: 'tel', text: '电话', dictCode: '' })
      fieldList.push({ type: 'string', value: 'email', text: '邮箱', dictCode: '' })
      fieldList.push({ type: 'string', value: 'fax', text: '传真', dictCode: '' })
      fieldList.push({ type: 'string', value: 'taxRegistrationNumber', text: '税务登记号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'address', text: '地址', dictCode: '' })
      fieldList.push({ type: 'string', value: 'zipCode', text: '邮编', dictCode: '' })
      fieldList.push({ type: 'string', value: 'openingBank', text: '开户行', dictCode: '' })
      fieldList.push({ type: 'string', value: 'bankAccount', text: '银行帐户', dictCode: '' })
      fieldList.push({ type: 'string', value: 'website', text: '网址', dictCode: '' })
      fieldList.push({ type: 'string', value: 'note', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
