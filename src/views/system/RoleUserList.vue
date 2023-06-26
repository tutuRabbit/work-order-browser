<template>
  <a-row :gutter="10">
    <a-col :md="leftColMd" :sm="24" style="margin-bottom: 20px">
      <a-card :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <!-- 搜索区域 -->
          <a-form layout="inline" @keyup.enter.native="searchQuery">
            <a-row :gutter="24">
              <a-col :span="8">
                <a-form-item label="角色名称">
                  <!-- <a-form-item label="角色名称" :labelCol="{ span: 3 }" :wrapperCol="{ span: 18, offset: 1 }"> -->
                  <j-input placeholder="输入角色名称模糊查询" v-model="queryParam.roleName"></j-input>
                </a-form-item>
              </a-col>
              <!--
              <a-col :md="11" :sm="12">
                <a-form-item label="创建时间" :labelCol="{span: 5}" :wrapperCol="{span: 18, offset: 1}">
                  <j-date v-model="queryParam.createTime_begin" :showTime="true" date-format="YYYY-MM-DD HH:mm:ss" style="width:45%" placeholder="请选择开始时间" ></j-date>
                  <span style="width: 10px;">~</span>
                  <j-date v-model="queryParam.createTime_end" :showTime="true" date-format="YYYY-MM-DD HH:mm:ss" style="width:45%" placeholder="请选择结束时间"></j-date>
                </a-form-item>
              </a-col>
              -->
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-col :md="12" :sm="24">
                  <a-button type="primary" @click="searchQuery" icon="search" style="margin-left: 21px">查询</a-button>
                  <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                </a-col>
              </span>
            </a-row>
          </a-form>
        </div>
        <!-- 操作按钮区域 -->
        <div class="table-operator" style="margin: 5px 0 10px 2px">
          <a-button @click="handleAdd" type="primary" icon="plus" v-has="'roleUser:add'">新建角色</a-button>
          <!--<a-button @click="handleEdit(model1)" type="primary" icon="plus">角色编辑</a-button>-->
          <!-- <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
            <a-button type="primary" icon="import">导入</a-button>
          </a-upload>
          <a-button type="primary" icon="download" @click="handleExportXls('角色管理')">导出</a-button> -->
        </div>

        <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
          <i class="anticon anticon-info-circle ant-alert-icon"> </i> 已选择
          <a
            ><b>{{ selectedRowKeys1.length }}</b></a
          >项
          <a style="margin-left: 24px" @click="onClearSelected1">清空</a>
        </div>

        <div style="margin-top: 15px">
          <a-table
            style="height:500px"
            ref="table"
            size="middle"
            bordered
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :loading="loading"
            :rowSelection="{ selectedRowKeys: selectedRowKeys1, onChange: onSelectChange1, type: 'radio' }"
            @change="handleTableChange"
          >
            <span slot="action" slot-scope="text, record">
              <a @click="handleOpen(record)">用户</a>
              <a-divider type="vertical" v-has="'roleUser:add'" />

              <a-dropdown v-has="'roleUser:setting'">
                <a class="ant-dropdown-link"> 权限配置 <a-icon type="down" /> </a>
                <a-menu slot="overlay">
                  <a-menu-item>
                    <a @click="handlePerssion(record.id, 'pc')">移动端权限</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a @click="handlePerssion(record.id, 'web')">web端权限</a>
                  </a-menu-item>
                </a-menu>
              </a-dropdown>
              <a-divider type="vertical" v-has="'roleUser:setting'" />
              <a @click="handleEdit(record)" v-has="'roleUser:edit'">编辑</a>
              <a-divider type="vertical" v-has="'roleUser:edit'" />
              <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete1(record.id)" v-has="'roleUser:delete'">
                <a>删除</a>
              </a-popconfirm>
              <!-- <a-dropdown>
                <a class="ant-dropdown-link"> 更多 <a-icon type="down" /> </a
                <a-menu slot="overlay">
                   <a-menu-item>
                    <a @click="handlePerssion(record.id)">授权</a>
                  </a-menu-item> 
                  <a-menu-item>
                    <a @click="handleEdit(record)">编辑</a>
                  </a-menu-item>
                  <a-menu-item>
                    <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete1(record.id)">
                      <a>删除</a>
                    </a-popconfirm>
                  </a-menu-item>
                </a-menu>
              </a-dropdown> -->
            </span>
          </a-table>
        </div>
        <!-- 右侧的角色权限配置 -->
        <user-role-modal ref="modalUserRole"></user-role-modal>
        <role-modal ref="modalForm" @ok="modalFormOk"></role-modal>
        <!-- 移动端权限 -->
        <j-modal
          title="移动端权限"
          :width="700"
          :visible="PCVisible"
          @ok="handlePCOk"
          @cancel="handlePCCancel"
          cancelText="关闭"
        >
          <!-- 查询区域 -->
          <div class="table-page-search-wrapper">
            <a-form layout="inline">
              <a-row :gutter="24">
                <a-col :span="24">
                  <a-form-item label="工单 ">
                    <a-select
                      v-model="appParam.workOrder"
                      @change="PCChange(appParam.workOrder, '工单')"
                      placeholder="请选择"
                    >
                      <a-select-option value="0">全部权限</a-select-option>
                      <a-select-option value="1">部分权限</a-select-option>
                      <a-select-option value="2">没有权限</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="24" v-if="appParam.workOrder == '1'">
                  <a-select
                    mode="multiple"
                    placeholder="请选择"
                    v-model="selectParam.workOrder"
                    optionFilterProp="label"
                    style="padding: 0px 0px 10px 50px;width: 100%"
                  >
                    <a-select-option v-for="(item, index) in workOrderList" :key="index" :value="item.key">{{
                      item.slotTitle
                    }}</a-select-option>
                  </a-select>
                </a-col>
                <a-col :span="24">
                  <a-form-item label="任务 ">
                    <a-select v-model="appParam.task" placeholder="请选择" @change="PCChange(appParam.task, '任务')">
                      <a-select-option value="0">全部权限</a-select-option>
                      <a-select-option value="1">部分权限</a-select-option>
                      <a-select-option value="2">没有权限</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="24" v-if="appParam.task == '1'">
                  <a-select
                    mode="multiple"
                    placeholder="请选择"
                    v-model="selectParam.task"
                    optionFilterProp="label"
                    style="padding: 0px 0px 10px 50px;width: 100%"
                  >
                    <a-select-option v-for="(item, index) in taskList" :key="index" :value="item.key">{{
                      item.slotTitle
                    }}</a-select-option>
                  </a-select>
                </a-col>
                <a-col :span="24">
                  <a-form-item label="报工 ">
                    <a-select
                      v-model="appParam.workReporting"
                      placeholder="请选择"
                      @change="PCChange(appParam.workReporting, '报工')"
                    >
                      <a-select-option value="0">全部权限</a-select-option>
                      <a-select-option value="1">部分权限</a-select-option>
                      <a-select-option value="2">没有权限</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="24" v-if="appParam.workReporting == '1'">
                  <a-select
                    mode="multiple"
                    placeholder="请选择"
                    v-model="selectParam.workReporting"
                    optionFilterProp="label"
                    style="padding: 0px 0px 10px 50px;width: 100%"
                  >
                    <a-select-option v-for="(item, index) in workReportingList" :key="index" :value="item.key">{{
                      item.slotTitle
                    }}</a-select-option>
                  </a-select>
                </a-col>
                <a-col :span="24">
                  <a-form-item label="看板 ">
                    <a-select v-model="appParam.board" placeholder="请选择" @change="PCChange(appParam.board, '看板')">
                      <a-select-option value="0">全部权限</a-select-option>
                      <a-select-option value="1">部分权限</a-select-option>
                      <a-select-option value="2">没有权限</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="24" v-if="appParam.board == '1'">
                  <a-select
                    mode="multiple"
                    placeholder="请选择"
                    v-model="selectParam.board"
                    optionFilterProp="label"
                    style="padding: 0px 0px 10px 50px;width: 100%"
                  >
                    <a-select-option v-for="(item, index) in boardList" :key="index" :value="item.key">{{
                      item.slotTitle
                    }}</a-select-option>
                  </a-select>
                </a-col>
                <a-col :span="24">
                  <a-form-item label="个人 ">
                    <a-select
                      v-model="appParam.personage"
                      placeholder="请选择"
                      @change="PCChange(appParam.personage, '个人')"
                    >
                      <a-select-option value="0">全部权限</a-select-option>
                      <a-select-option value="1">部分权限</a-select-option>
                      <a-select-option value="2">没有权限</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :span="24" v-if="appParam.personage == '1'">
                  <a-select
                    mode="multiple"
                    placeholder="请选择"
                    v-model="selectParam.personage"
                    optionFilterProp="label"
                    style="padding: 0px 0px 10px 50px;width: 100%"
                  >
                    <a-select-option v-for="(item, index) in personageList" :key="index" :value="item.key">{{
                      item.slotTitle
                    }}</a-select-option>
                  </a-select>
                </a-col>
              </a-row>
            </a-form>
          </div>
        </j-modal>
      </a-card>
    </a-col>
    <a-col :md="rightColMd" :sm="24" v-if="this.rightcolval == 1">
      <a-card :bordered="false">
        <div style="text-align: right;">
          <a-icon type="close-circle" @click="hideUserList" />
        </div>
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline">
            <a-row :gutter="24">
              <a-col :md="12" :sm="12">
                <a-form-item label="用户账号">
                  <a-input placeholder="" v-model="queryParam2.username"></a-input>
                </a-form-item>
              </a-col>
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-col :md="9" :sm="24">
                  <a-button type="primary" @click="searchQuery2" icon="search" style="margin-left: 21px">查询</a-button>
                  <a-button type="primary" @click="searchReset2" icon="reload" style="margin-left: 8px">重置</a-button>
                </a-col>
              </span>
            </a-row>
          </a-form>
        </div>
        <!-- 操作按钮区域 -->
        <div class="table-operator" :md="24" :sm="24">
          <a-button @click="handleAdd2" type="primary" icon="plus" style="margin-top: 16px">新增用户</a-button>
          <!--<a-button @click="handleEdit2" type="primary" icon="edit" style="margin-top: 16px">用户编辑</a-button>-->
          <a-button @click="handleAddUserRole" type="primary" icon="plus" style="margin-top: 16px">已有用户</a-button>

          <a-dropdown v-if="selectedRowKeys2.length > 0">
            <a-menu slot="overlay">
              <a-menu-item key="1" @click="batchDel2">
                <a-icon type="delete" />
                删除
              </a-menu-item>
            </a-menu>
            <a-button style="margin-left: 8px">
              批量操作
              <a-icon type="down" />
            </a-button>
          </a-dropdown>
        </div>
        <!-- table区域-begin -->
        <div>
          <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
            <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
            <a style="font-weight: 600">{{ selectedRowKeys2.length }}</a
            >项
            <a style="margin-left: 24px" @click="onClearSelected2">清空</a>
          </div>
          <a-table
            style="height:500px"
            ref="table2"
            bordered
            size="middle"
            rowKey="id"
            :columns="columns2"
            :dataSource="dataSource2"
            :pagination="ipagination2"
            :loading="loading2"
            :rowSelection="{ selectedRowKeys: selectedRowKeys2, onChange: onSelectChange2 }"
            @change="handleTableChange2"
          >
            <span slot="action" slot-scope="text, record">
              <a @click="handleEdit2(record)">编辑</a>
              <a-divider type="vertical" />
              <a-dropdown>
                <a class="ant-dropdown-link"> 更多 <a-icon type="down" /> </a>
                <a-menu slot="overlay">
                  <a-menu-item>
                    <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete2(record.id)">
                      <a>删除</a>
                    </a-popconfirm>
                  </a-menu-item>
                </a-menu>
              </a-dropdown>
            </span>
          </a-table>
        </div>
        <!-- 表单区域 -->
        <role-modal ref="modalForm" @ok="modalFormOk"></role-modal>
        <user-modal ref="modalForm2" @ok="modalFormOk2"></user-modal>
        <Select-User-Modal ref="selectUserModal" @selectFinished="selectOK"></Select-User-Modal>
      </a-card>
    </a-col>
  </a-row>
</template>
<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { deleteAction, postAction, getAction } from '@/api/manage'
import SelectUserModal from './modules/SelectUserModal'
import RoleModal from './modules/RoleModal'
import { saveRolePermission } from '@/api/api'
import UserModal from './modules/UserModal'
import { filterObj } from '@/utils/util'
import UserRoleModal from './modules/UserRoleModal'
import moment from 'moment'

export default {
  name: 'RoleUserList',
  mixins: [JeecgListMixin],
  components: {
    UserRoleModal,
    SelectUserModal,
    RoleModal,
    UserModal,
    moment
  },
  data() {
    return {
      // 移动端权限
      PCVisible: false,
      PCParam: {},
      appParam: {
        workOrder: '',
        task: '',
        workReporting: '',
        board: '',
        personage: ''
      },
      selectParam: {
        workOrder: [],
        task: [],
        workReporting: [],
        board: [],
        personage: []
      },
      roleId: '',
      lastpermissionIds: [],
      // end
      model1: {},
      model2: {},
      currentRoleId: '',
      queryParam1: {},
      queryParam2: {},
      dataSource1: [],
      dataSource2: [],
      ipagination1: {
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      ipagination2: {
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      isorter1: {
        column: 'createTime',
        order: 'desc'
      },
      isorter2: {
        column: 'createTime',
        order: 'desc'
      },
      filters1: {},
      filters2: {},
      loading1: false,
      loading2: false,
      selectedRowKeys1: [],
      selectedRowKeys2: [],
      selectionRows1: [],
      selectionRows2: [],
      test: {},
      rightcolval: 0,
      columns: [
        {
          title: '角色编码',
          align: 'center',
          dataIndex: 'roleCode'
        },
        {
          title: '角色名称',
          align: 'center',
          dataIndex: 'roleName'
        },
        {
          title: '创建时间',
          dataIndex: 'createTime',
          align: 'center',
          sorter: true,
          customRender: text => {
            return moment(text).format('YYYY-MM-DD')
          }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      columns2: [
        {
          title: '用户账号',
          align: 'center',
          dataIndex: 'username',
          width: 120
        },
        {
          title: '用户名称',
          align: 'center',
          width: 100,
          dataIndex: 'realname'
        },
        {
          title: '状态',
          align: 'center',
          width: 80,
          dataIndex: 'status_dictText'
        },

        {
          title: '操作',
          dataIndex: 'action',
          scopedSlots: { customRender: 'action' },
          align: 'center',
          width: 120
        }
      ],

      // 高级查询参数
      superQueryParams2: '',
      // 高级查询拼接条件
      superQueryMatchType2: 'and',
      treeList: [],
      workOrderList: [], //工单
      taskList: [], //任务
      workReportingList: [], //报工
      boardList: [], //看板
      personageList: [], //个人
      url: {
        list: '/sys/role/list',
        delete: '/sys/role/delete',
        list2: '/sys/user/userRoleList',
        addUserRole: '/sys/user/addSysUserRole',
        delete2: '/sys/user/deleteUserRole',
        deleteBatch2: '/sys/user/deleteUserRoleBatch',
        exportXlsUrl: 'sys/role/exportXls',
        importExcelUrl: 'sys/role/importExcel',
        queryAppTreeList: '/sys/role/queryAppTreeList?roleId=' //app角色权限List
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    },
    leftColMd() {
      return this.selectedRowKeys1.length === 0 ? 24 : 12
    },
    rightColMd() {
      return this.selectedRowKeys1.length === 0 ? 0 : 12
    }
  },
  methods: {
    PCChange(param, type) {
      // console.log('param', param)
      // console.log('type', type)
      if (type == '工单') {
        if (param == '0') {
          this.selectParam.workOrder = this.workOrderList.map(item => item.key)
        } else if (param == '2') {
          this.selectParam.workOrder = []
        }
      }
      //
      else if (type == '任务') {
        if (param == '0') {
          this.selectParam.task = this.taskList.map(item => item.key)
        } else if (param == '2') {
          this.selectParam.task = []
        }
      }
      //
      else if (type == '报工') {
        if (param == '0') {
          this.selectParam.workReporting = this.workReportingList.map(item => item.key)
        } else if (param == '2') {
          this.selectParam.workReporting = []
        }
      }
      //
      else if (type == '看板') {
        if (param == '0') {
          this.selectParam.board = this.boardList.map(item => item.key)
        } else if (param == '2') {
          this.selectParam.board = []
        }
      }
      //
      else if (type == '个人') {
        if (param == '0') {
          this.selectParam.personage = this.personageList.map(item => item.key)
        } else if (param == '2') {
          this.selectParam.personage = []
        }
      }
      this.$forceUpdate()
    },
    handlePCCancel() {
      this.appParam = {}
      this.selectParam = {
        workOrder: [],
        task: [],
        workReporting: [],
        board: [],
        personage: []
      }
      this.PCVisible = false
    },
    onSelectChange2(selectedRowKeys, selectionRows) {
      this.selectedRowKeys2 = selectedRowKeys
      this.selectionRows2 = selectionRows
    },
    onClearSelected2() {
      this.selectedRowKeys2 = []
      this.selectionRows2 = []
    },
    onClearSelected1() {
      this.selectedRowKeys1 = []
      this.selectionRows1 = []
    },
    onSelectChange1(selectedRowKeys, selectionRows) {
      this.rightcolval = 1
      this.selectedRowKeys1 = selectedRowKeys
      this.selectionRows1 = selectionRows
      this.model1 = Object.assign({}, selectionRows[0])
      console.log(this.model1)
      this.currentRoleId = selectedRowKeys[0]
      this.loadData2()
    },
    onClearSelected() {},

    getQueryParams2() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams2) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams2)
        sqp['superQueryMatchType'] = this.superQueryMatchType2
      }
      var param = Object.assign(sqp, this.queryParam2, this.isorter2, this.filters2)
      param.field = this.getQueryField2()
      param.pageNo = this.ipagination2.current
      param.pageSize = this.ipagination2.pageSize
      return filterObj(param)
    },
    getQueryField2() {
      //TODO 字段权限控制
      var str = 'id,'
      this.columns2.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    handleEdit2: function(record) {
      this.$refs.modalForm2.title = '编辑'
      this.$refs.modalForm2.roleDisabled = true
      this.$refs.modalForm2.edit(record)
    },
    handleAdd2: function() {
      if (this.currentRoleId == '') {
        this.$message.error('请选择一个角色!')
      } else {
        this.$refs.modalForm2.roleDisabled = true
        this.$refs.modalForm2.title = '新增'
        this.$refs.modalForm2.edit({ activitiSync: '1', userIdentity: 1, selectedroles: this.currentRoleId })
      }
    },
    modalFormOk2() {
      // 新增/修改 成功时，重载列表
      this.loadData2()
    },
    loadData2(arg) {
      if (!this.url.list2) {
        this.$message.error('请设置url.list2属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination2.current = 1
      }
      if (this.currentRoleId === '') return
      let params = this.getQueryParams2() //查询条件
      params.roleId = this.currentRoleId
      this.loading2 = true
      getAction(this.url.list2, params).then(res => {
        if (res.success) {
          this.dataSource2 = res.result.records
          this.ipagination2.total = res.result.total
        }
        this.loading2 = false
      })
    },
    handleDelete1: function(id) {
      this.handleDelete(id)
      this.dataSource2 = []
      this.currentRoleId = ''
    },
    handleDelete2: function(id) {
      if (!this.url.delete2) {
        this.$message.error('请设置url.delete2属性!')
        return
      }
      var that = this
      deleteAction(that.url.delete2, { roleId: this.currentRoleId, userId: id }).then(res => {
        if (res.success) {
          that.$message.success(res.message)
          that.loadData2()
        } else {
          that.$message.warning(res.message)
        }
      })
    },
    batchDel2: function() {
      if (!this.url.deleteBatch2) {
        this.$message.error('请设置url.deleteBatch2属性!')
        return
      }
      if (this.selectedRowKeys2.length <= 0) {
        this.$message.warning('请选择一条记录！')
        return
      } else {
        var ids = ''
        for (var a = 0; a < this.selectedRowKeys2.length; a++) {
          ids += this.selectedRowKeys2[a] + ','
        }
        var that = this
        console.log(this.currentDeptId)
        this.$confirm({
          title: '确认删除',
          content: '是否删除选中数据?',
          onOk: function() {
            deleteAction(that.url.deleteBatch2, { roleId: that.currentRoleId, userIds: ids }).then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.loadData2()
                that.onClearSelected()
              } else {
                that.$message.warning(res.message)
              }
            })
          }
        })
      }
    },
    selectOK(data) {
      let params = {}
      params.roleId = this.currentRoleId
      params.userIdList = []
      for (var a = 0; a < data.length; a++) {
        params.userIdList.push(data[a])
      }
      console.log(params)
      postAction(this.url.addUserRole, params).then(res => {
        if (res.success) {
          this.loadData2()
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
      })
    },

    handleAddUserRole() {
      if (this.currentRoleId == '') {
        this.$message.error('请选择一个角色!')
      } else {
        this.$refs.selectUserModal.visible = true
      }
    },
    handleOpen(record) {
      this.rightcolval = 1
      this.selectedRowKeys1 = [record.id]
      this.model1 = Object.assign({}, record)
      this.currentRoleId = record.id
      this.onClearSelected2()
      this.loadData2()
    },
    /*handleEdit: function(record) {
        if (this.currentRoleId == '') {
          this.$message.error('请选择一个角色!')
        } else {
          this.$refs.modalForm.edit(record)
          this.$refs.modalForm.title = '编辑'
        }
      },*/
    searchQuery2() {
      this.loadData2(1)
    },
    searchReset2() {
      this.queryParam2 = {}
      this.loadData2(1)
    },
    handleTableChange2(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter2.column = sorter.field
        this.isorter2.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination2 = pagination
      this.loadData2()
    },
    hideUserList() {
      //this.rightcolval = 0
      this.selectedRowKeys1 = []
    },
    handlePerssion(roleId, val) {
      let that = this
      if (val == 'web') {
        that.$refs.modalUserRole.show(roleId)
      } else {
        that.roleId = roleId
        getAction(that.url.queryAppTreeList + roleId).then(res => {
          let treeList = res.result.treeList
          that.lastpermissionIds = res.result.ids
          // console.log('  that.lastpermissionIds', that.lastpermissionIds)
          treeList.forEach(item => {
            if (item.slotTitle == 'APP') {
              that.treeList = item.children
            }
          })

          if (that.treeList.length != 0) {
            that.treeList.forEach(item => {
              if (item.slotTitle == '工单') {
                that.workOrderList = item.children
              }
              if (item.slotTitle == '任务') {
                that.taskList = item.children
              }
              if (item.slotTitle == '报工') {
                that.workReportingList = item.children
              }
              if (item.slotTitle == '看板') {
                that.boardList = item.children
              }
              if (item.slotTitle == '个人') {
                that.personageList = item.children
              }
            })
          }
          console.log('that.lastpermissionIds', that.lastpermissionIds)
          // if (that.lastpermissionIds.length != 0) {
          //工单
          that.selectParam.workOrder = this.filterList(that.workOrderList)
          that.appParam.workOrder = that.listLength(that.selectParam.workOrder, that.workOrderList)
          //任务
          that.selectParam.task = this.filterList(that.taskList)
          that.appParam.task = that.listLength(that.selectParam.task, that.taskList)
          //报工
          that.selectParam.workReporting = this.filterList(that.workReportingList)
          that.appParam.workReporting = that.listLength(that.selectParam.workReporting, that.workReportingList)
          //看板
          that.selectParam.board = this.filterList(that.boardList)
          that.appParam.board = that.listLength(that.selectParam.board, that.boardList)
          // 个人
          that.selectParam.personage = this.filterList(that.personageList)
          that.appParam.personage = that.listLength(that.selectParam.personage, that.personageList)
          // }
        })
        console.log(' that.appParam', that.appParam)
        setTimeout(() => {
          that.$forceUpdate()
          that.PCVisible = true
        }, 500)
      }
    },
    listLength(a, b) {
      if (a.length === b.length) {
        return '0'
      } else if (a.length != b.length && a.length != 0 && b.length != 0) {
        return '1'
      } else if (a.length == '0') {
        return '2'
      }
    },
    filterList(arr) {
      return this.lastpermissionIds.filter(item => arr.some(ele => ele.key === item))
    },
    handlePCOk() {
      let that = this
      let a = this.selectParam.workOrder.length != 0 ? this.selectParam.workOrder.toString() : ''
      let b = this.selectParam.task.length != 0 ? this.selectParam.task.toString() : ''
      let c = this.selectParam.workReporting.length != 0 ? this.selectParam.workReporting.toString() : ''
      let d = this.selectParam.board.length != 0 ? this.selectParam.board.toString() : ''
      let e = this.selectParam.personage.length != 0 ? this.selectParam.personage.toString() : ''
      var permissionIds = a + ',' + b + ',' + c + ',' + d + ',' + e
      let params = {
        roleId: that.roleId,
        permissionIds: permissionIds,
        lastpermissionIds: that.lastpermissionIds.toString()
      }
      console.log('params', params)
      saveRolePermission(params).then(res => {
        if (res.success) {
          that.$message.success(res.message)
        } else {
          that.$message.error(res.message)
        }
      })
      this.loadData()
      this.handlePCCancel()
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
/** Button按钮间距 */
.ant-btn {
  margin-left: 8px;
}
::v-deep {
  .table-page-search-wrapper .ant-form-inline .ant-form-item {
    margin-bottom: 8px;
  }
}
</style>
