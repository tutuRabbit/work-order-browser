<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('出入库明细表1')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text,record">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" :preview="record.id" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
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

    <sys-library-detailed-modal ref="modalForm" @ok="modalFormOk"></sys-library-detailed-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import SysLibraryDetailedModal from './modules/SysLibraryDetailedModal'

  export default {
    name: 'SysLibraryDetailedList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      SysLibraryDetailedModal
    },
    data () {
      return {
        description: '出入库明细表1管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'单号',
            align:"center",
            dataIndex: 'code'
          },
          {
            title:'出入库标识',
            align:"center",
            dataIndex: 'type'
          },
          {
            title:'产品ID',
            align:"center",
            dataIndex: 'productId'
          },
          {
            title:'产品编码',
            align:"center",
            dataIndex: 'productCode'
          },
          {
            title:'出入库数量',
            align:"center",
            dataIndex: 'libraryNumber'
          },
          {
            title:'运费',
            align:"center",
            dataIndex: 'freight'
          },
          {
            title:'客户ID',
            align:"center",
            dataIndex: 'customer'
          },
          {
            title:'客户名称',
            align:"center",
            dataIndex: 'customerName'
          },
          {
            title:'产品规格',
            align:"center",
            dataIndex: 'lastProductSpec'
          },
          {
            title:'单位ID',
            align:"center",
            dataIndex: 'lastProductUnitId'
          },
          {
            title:'单位',
            align:"center",
            dataIndex: 'lastProductUnitName'
          },
          {
            title:'创建时间',
            align:"center",
            dataIndex: 'createTime'
          },
          {
            title:'是否删除',
            align:"center",
            dataIndex: 'deletedAt'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/longxianda/sysLibraryDetailed/list",
          delete: "/longxianda/sysLibraryDetailed/delete",
          deleteBatch: "/longxianda/sysLibraryDetailed/deleteBatch",
          exportXlsUrl: "/longxianda/sysLibraryDetailed/exportXls",
          importExcelUrl: "longxianda/sysLibraryDetailed/importExcel",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'code',text:'单号',dictCode:''})
        fieldList.push({type:'string',value:'type',text:'出入库标识',dictCode:''})
        fieldList.push({type:'string',value:'productId',text:'产品ID',dictCode:''})
        fieldList.push({type:'string',value:'productCode',text:'产品编码',dictCode:''})
        fieldList.push({type:'string',value:'libraryNumber',text:'出入库数量',dictCode:''})
        fieldList.push({type:'string',value:'freight',text:'运费',dictCode:''})
        fieldList.push({type:'string',value:'customer',text:'客户ID',dictCode:''})
        fieldList.push({type:'string',value:'customerName',text:'客户名称',dictCode:''})
        fieldList.push({type:'string',value:'lastProductSpec',text:'产品规格',dictCode:''})
        fieldList.push({type:'string',value:'lastProductUnitId',text:'单位ID',dictCode:''})
        fieldList.push({type:'string',value:'lastProductUnitName',text:'单位',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建时间'})
        fieldList.push({type:'string',value:'deletedAt',text:'是否删除',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>