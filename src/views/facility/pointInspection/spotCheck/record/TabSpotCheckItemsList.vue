<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="计划所属部门">
              <a-tree-select
                v-model.trim="queryParam.planDepartment"
                style="width: 100%"
                :tree-data="treeData"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                placeholder="请选择"
              >
              </a-tree-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="设备巡检情况">
              <j-dict-select-tag
                dictCode="equipment_inspection"
                v-model.trim="queryParam.equipmentCheckStatus"
              ></j-dict-select-tag>
              <!-- <a-input v-model.trim="queryParam.equipmentCheckStatus" placeholder="请输入"> </a-input> -->
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="计划名称">
                <a-input v-model.trim="queryParam.planName" placeholder="请输入"> </a-input>
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
      <a-button type="primary" icon="import" @click="handleExport('巡检记录')">导出</a-button>
      <a-button type="primary">打印</a-button>
      <!-- <a-button @click="handleAdd" type="primary">跳过巡检</a-button> -->
      <a-button type="primary" @click="handleJump">跳过巡检</a-button>
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
        <span slot="action" slot-scope="text, record">
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
          <!-- <a @click="handleEdit(record)">编辑</a>
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
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <div class="title">设备情况</div>
    <Check-record-List @ok="loadData" ref="checkRecordList"></Check-record-List>
    <!--新增、编辑 -->
    <tab-spot-check-items-modal ref="modalForm" @ok="modalFormOk"></tab-spot-check-items-modal>
    <!-- 跳过巡检 -->
    <j-modal title="跳过巡检" width="600px" :visible="jumpVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="跳过原因">
                <a-select
                  v-model.trim="jumpParam.skipReason"
                  :showSearch="true"
                  optionFilterProp="label"
                  style="width: 100%"
                  placeholder="请选择"
                >
                  <a-select-option value="0" label="保养中">保养中</a-select-option>
                  <a-select-option value="1" label="外委中">外委中</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="loading" type="primary" @click="jumpOk">确认</a-button>
      </template>
    </j-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { getAction, downFile } from '@/api/manage'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import TabSpotCheckItemsModal from './modules/TabSpotCheckItemsModal'
import CheckRecordList from './modules/checkRecordList'
import { queryDepartTreeList } from '@/api/api'

export default {
  name: 'TabSpotCheckItemsList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    CheckRecordList,
    TabSpotCheckItemsModal
  },
  data() {
    return {
      treeData: [],
      description: '巡检记录管理页面',
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
          title: '计划名称',
          align: 'center',
          dataIndex: 'planName'
        },
        {
          title: '设备巡检进度',
          align: 'center',
          dataIndex: 'patrolInspectionProgress'
        },
        {
          title: '计划巡检时间',
          align: 'center'
          // dataIndex: 'planStartTime'planEndTime//两个字段拼起来
        },
        {
          title: '巡检开始时间',
          align: 'center',
          dataIndex: 'checkStartTime'
        },
        {
          title: '巡检结束时间',
          align: 'center',
          dataIndex: 'checkEndTime'
        },
        {
          title: '计划巡检人',
          align: 'center',
          dataIndex: 'planExecutor'
        },
        {
          title: '计划所属部门',
          align: 'center',
          dataIndex: 'planDepartment'
        },
        {
          title: '计划跳过原因',
          align: 'center',
          dataIndex: 'skipReason'
        },
        {
          title: '跳过人',
          align: 'center',
          dataIndex: 'skipOperator'
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
        list: '/checkplan/patrolInspectionHistory/list',
        delete: '/checkplan/patrolInspectionHistory/delete',
        deleteBatch: '/checkplan/patrolInspectionHistory/deleteMany',
        exportXlsUrl: '/checkplan/patrolInspectionHistory/export',
        skip: '/checkplan/patrolInspectionHistory/skip', //跳过任务
        importExcelUrl: 'checkplan/patrolInspectionHistory/importExcel'
      },
      dictOptions: {},
      jumpVisible: false,
      jumpParam: {},
      superFieldList: []
    }
  },
  created() {
    // 这个方法是找到所有的部门信息
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
    handleExport(fileName) {
      if (this.selectedRowKeys.length != 0) {
        if (!fileName || typeof fileName != 'string') {
          fileName = '导出文件'
        }
        let param = this.getQueryParams()
        if (this.selectedRowKeys && this.selectedRowKeys.length > 0) {
          param['ids'] = this.selectedRowKeys.join(',')
          // param['selections'] = this.selectedRowKeys.join(',')
        }
        console.log('导出参数', param)
        downFile(this.url.exportXlsUrl, param).then(data => {
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
        this.$confirm({
          title: '确认',
          content: '当前未选择巡检记录,是否导出所有记录?',
          onOk: () => {
            //
            if (!fileName || typeof fileName != 'string') {
              fileName = '导出文件'
            }
            let param = this.getQueryParams()
            if (this.selectedRowKeys && this.selectedRowKeys.length > 0) {
              param['ids'] = this.selectedRowKeys.join(',')
            }
            console.log('导出参数', param)
            downFile(this.url.exportXlsUrl, param).then(data => {
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
          }
        })
      }
    },
    handleJump() {
      if (this.selectedRowKeys.length == 1) {
        this.jumpParam.id = this.selectionRows[0].id
        this.jumpVisible = true
      } else {
        this.$message.warning('请选中一条数据！')
      }
    },
    handleCancel() {
      this.jumpVisible = false
      this.jumpParam = {}
    },
    jumpOk() {
      let pm = new URLSearchParams()
      pm.append('id', this.jumpParam.id)
      pm.append('skipReason', this.jumpParam.skipReason)
      console.log('pm', pm)

      getAction(this.url.skip + '?id=' + this.jumpParam.id + '&' + 'skipReason=' + this.jumpParam.skipReason).then(
        res => {
          if (res.success) {
            if (res.code == 200) {
              this.$message.success(res.message)
              this.loadData()
            }
          } else {
            this.$message.warning(res.message)
          }
          this.handleCancel()
        }
      )
    },
    //
    handleEdit: function(record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '编辑'
      this.$refs.modalForm.disableSubmit = false
    },
    handleTableClick(record, index) {
      return {
        on: {
          click: () => {
            this.$refs.checkRecordList.details(record)
          },
          dblclick: () => {
            console.log('双击了我')
          }
        }
      }
    },
    onChangeRangeDate(value, dateString) {
      this.queryParam.lastSpotCheckStartTime = dateString[0]
      this.queryParam.lastSpotCheckEndTime = dateString[1]
      console.log('this.queryParam.lastSpotCheckEndTime', this.queryParam)
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
