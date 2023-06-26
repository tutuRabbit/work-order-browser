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
      <a-button type="primary" icon="download" @click="handleExportXls('工单明细表')">导出</a-button>
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

    <sys-work-order-detail-modal ref="modalForm" @ok="modalFormOk"></sys-work-order-detail-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import SysWorkOrderDetailModal from './modules/SysWorkOrderDetailModal'

  export default {
    name: 'SysWorkOrderDetailList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      SysWorkOrderDetailModal
    },
    data () {
      return {
        description: '工单明细表管理页面',
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
            title:'工序编号',
            align:"center",
            dataIndex: 'procedureCode'
          },
          {
            title:'工序名称',
            align:"center",
            dataIndex: 'procedureName'
          },
          {
            title:'报工权限',
            align:"center",
            dataIndex: 'outputUsers'
          },
          {
            title:'报工数配比',
            align:"center",
            dataIndex: 'outputRate'
          },
          {
            title:'不良品列表',
            align:"center",
            dataIndex: 'defects'
          },
          {
            title:'计划数',
            align:"center",
            dataIndex: 'planNumber'
          },
          {
            title:'良品数',
            align:"center",
            dataIndex: 'goodsPros'
          },
          {
            title:'不良品数',
            align:"center",
            dataIndex: 'rejects'
          },
          {
            title:'计划开始时间',
            align:"center",
            dataIndex: 'planStartTime'
          },
          {
            title:'计划结束时间',
            align:"center",
            dataIndex: 'planEntTime'
          },
          {
            title:'开始时间',
            align:"center",
            dataIndex: 'startTime'
          },
          {
            title:'结束时间',
            align:"center",
            dataIndex: 'endTime'
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
          list: "/longxianda/sysWorkOrderDetail/list",
          delete: "/longxianda/sysWorkOrderDetail/delete",
          deleteBatch: "/longxianda/sysWorkOrderDetail/deleteBatch",
          exportXlsUrl: "/longxianda/sysWorkOrderDetail/exportXls",
          importExcelUrl: "longxianda/sysWorkOrderDetail/importExcel",
          
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
        fieldList.push({type:'string',value:'procedureCode',text:'工序编号',dictCode:''})
        fieldList.push({type:'string',value:'procedureName',text:'工序名称',dictCode:''})
        fieldList.push({type:'string',value:'outputUsers',text:'报工权限',dictCode:''})
        fieldList.push({type:'double',value:'outputRate',text:'报工数配比',dictCode:''})
        fieldList.push({type:'string',value:'defects',text:'不良品列表',dictCode:''})
        fieldList.push({type:'int',value:'planNumber',text:'计划数',dictCode:''})
        fieldList.push({type:'int',value:'goodsPros',text:'良品数',dictCode:''})
        fieldList.push({type:'int',value:'rejects',text:'不良品数',dictCode:''})
        fieldList.push({type:'datetime',value:'planStartTime',text:'计划开始时间'})
        fieldList.push({type:'datetime',value:'planEntTime',text:'计划结束时间'})
        fieldList.push({type:'datetime',value:'startTime',text:'开始时间'})
        fieldList.push({type:'datetime',value:'endTime',text:'结束时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>