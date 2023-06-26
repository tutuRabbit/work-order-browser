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
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'sysCustom:add'">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('自定义字段配置表2')" v-has="'sysCustom:export'"
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
        <a-button type="primary" icon="import" v-has="'sysCustom:import'">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'sysCustom:delete'">
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
          <a @click="handleEdit(record)" v-has="'sysCustom:edit'">编辑</a>
          <a-divider type="vertical" v-has="'sysCustom:edit'" />
          <a @click="handleDetail(record)" v-has="'sysCustom:view'">详情</a>
          <a-divider type="vertical" v-has="'sysCustom:view'" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'sysCustom:delete'">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)" v-has="'sysCustom:add'">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'sysCustom:add'">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>

    <sys-custom-modal ref="modalForm" @ok="modalFormOk"></sys-custom-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import SysCustomModal from './modules/SysCustomModal'

export default {
  name: 'SysCustomList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    SysCustomModal
  },
  data() {
    return {
      description: '自定义字段配置',
      // 表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '自定义类型',
          align: 'center',
          dataIndex: 'targetType',
          customRender: function(t, r, index) {
            if (r.targetType == '1001') {
              return '产品定义'
            } else if (r.targetType == '1002') {
              return '工序'
            } else if (r.targetType == '1003') {
              return '工艺线路'
            } else if (r.targetType == '1004') {
              return '不良品'
            } else if (r.targetType == '1005') {
              return '入库单'
            } else if (r.targetType == '1006') {
              return '出库单'
            } else if (r.targetType == '1007') {
              return '工单'
            }
          }
        },
        {
          title: '字段名称',
          align: 'center',
          dataIndex: 'name'
        },
        {
          title: '字段类型',
          align: 'center',
          dataIndex: 'type'
        },
        {
          title: '创建人名称',
          align: 'center',
          dataIndex: 'userName'
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
        list: '/longxianda/sysCustom/list',
        delete: '/longxianda/sysCustom/delete',
        deleteBatch: '/longxianda/sysCustom/deleteBatch',
        exportXlsUrl: '/longxianda/sysCustom/exportXls',
        importExcelUrl: 'longxianda/sysCustom/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    console.log(this.queryParam)
    this.getSuperFieldList()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    initDictConfig() {
      this.queryParam.targetYpe = '5995'
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'targetType', text: '自定义类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'name', text: '字段名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'type', text: '字段类型', dictCode: '' })

      fieldList.push({ type: 'string', value: 'userName', text: '创建人名称', dictCode: '' })

      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
