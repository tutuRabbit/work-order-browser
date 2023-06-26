<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">设备信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentCode">
            <a-input :disabled="true" v-model.trim="model.equipmentCode" placeholder="请输入设备编码"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="设备名称"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="equipmentName"
            :rules="[{ required: true, message: '请输入设备名称!' }]"
          >
            <a-input :disabled="true" v-model.trim="model.equipmentName" placeholder="请输入设备名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentType">
            <j-category-select
              :disabled="true"
              v-model.trim="model.equipmentType"
              pcode="B03"
              placeholder="请选择设备类型"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentModel">
            <a-input :disabled="true" v-model.trim="model.equipmentModel" placeholder="请输入规格型号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="所在部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="depart">
            <a-tree-select
              :disabled="true"
              v-model.trim="model.equipmentDepart"
              style="width: 100%"
              :tree-data="treeData"
              placeholder="请选择所在部门"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8" style="margin-left:7%">
          <a-button @click="selectEquipment" :disabled="formDisabled" type="primary">
            <a-icon type="search" /> 选择设备
          </a-button>
        </a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">外委信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="外委单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outFixEntrustNo">
            <a-input :disabled="formDisabled" v-model="model.outFixEntrustNo" placeholder="不填则自动生成"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="外委单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="entrustCompany">
            <a-input :disabled="formDisabled" v-model="model.entrustCompany" placeholder="请输入外委单位"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="entrustContact">
            <a-input :disabled="formDisabled" v-model="model.entrustContact" placeholder="请输入联系人"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="联系电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="entrustTel">
            <a-input :disabled="formDisabled" v-model="model.entrustTel" placeholder="请输入联系电话"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="预估费用" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="predictCost">
            <a-input :disabled="formDisabled" v-model="model.predictCost" placeholder="请输入预估费用"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="完成时限" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="completeTime">
            <j-date
              :disabled="formDisabled"
              placeholder="请选择完成时限"
              v-model="model.completeTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="申请原因"
            :rules="[{ required: true, message: '请输入申请原因!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="applyReason"
          >
            <a-input
              :disabled="formDisabled"
              type="textarea"
              v-model="model.applyReason"
              placeholder="请输入申请原因"
            ></a-input>
          </a-form-model-item>
        </a-col>
        <a-col v-if="executeT" :span="24">
          <a-col :span="8">
            <a-form-model-item label="实际费用" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="tureCost">
              <a-input v-model="model.tureCost" placeholder="请输入实际费用"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="外委状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <a-select v-model.trim="model.status" style="width: 100%">
                <a-select-option value="0">外委中</a-select-option>
                <a-select-option value="1">已完成</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修用时" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixSpendHour">
              <a-input
                style="width: 35%;margin-right: 20px"
                v-model.number="model.fixSpendHour"
                placeholder="请输入小时"
              ></a-input
              >小时
              <a-input
                style="width: 38%;margin-right: 20px"
                v-model.number="model.fixSpendMin"
                placeholder="请输入分"
              ></a-input
              >分
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item
              label="外委内容"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="applyReason"
              :rules="[{ required: model.status == '1' ? true : false, message: '请输入外委内容!' }]"
            >
              <a-input type="textarea" v-model="model.outEntrustContent" placeholder="请输入外委内容"></a-input>
            </a-form-model-item>
          </a-col>
        </a-col>
        <a-col v-if="verifyT" :span="24">
          <a-col>
            <div class="title" style="margin-top:20px ">验证</div>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="是否修复" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="repair">
              <a-radio-group v-model="model.repair" :disabled="formDisabled">
                <a-radio value="0">是</a-radio>
                <a-radio value="1">否</a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item
              label="验证说明"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              :rules="[{ required: true, message: '请输入验证说明!' }]"
            >
              <a-input v-model="model.verificationDescription" placeholder="请输入验证说明"></a-input>
            </a-form-model-item>
          </a-col>
        </a-col>
      </a-row>
    </a-form-model>
    <div class="title" v-if="!verifyT" style="margin-top:20px ">关联文档</div>
    <a-col :span="24" v-if="!verifyT">
      <a-col :span="24">
        <a-form-item
          label="选择文件"
          :labelCol="{
            xs: { span: 24 },
            sm: { span: 2 }
          }"
          :wrapperCol="{
            xs: { span: 24 },
            sm: { span: 6 }
          }"
        >
          <a-upload
            name="file"
            :multiple="true"
            :fileList="fileList"
            :remove="handleRemove"
            :beforeUpload="beforeUpload"
          >
            <a-button :disabled="formDisabled" type="info" icon="upload">选择文件</a-button>
          </a-upload>
        </a-form-item>
      </a-col>
      <a-col :span="24">
        <a-form-item
          label="上传提示"
          :labelCol="{
            xs: { span: 24 },
            sm: { span: 2 }
          }"
          :wrapperCol="{
            xs: { span: 24 },
            sm: { span: 6 }
          }"
        >
          <span>{{ queryParam.title }}</span>
        </a-form-item>
      </a-col>
      <a-col :span="24">
        <a-form-item
          label="文档类型"
          :labelCol="{
            xs: { span: 24 },
            sm: { span: 2 }
          }"
          :wrapperCol="{
            xs: { span: 24 },
            sm: { span: 6 }
          }"
        >
          <j-dict-select-tag
            :disabled="true"
            style="width:100%"
            type="list"
            v-model="queryParam.documentType"
            dictCode="doc_type"
            placeholder="请选择文档类型"
          />
        </a-form-item>
      </a-col>
    </a-col>
    <!-- 添加人员 -->
    <j-modal
      title="添加人员"
      width="60%"
      :visible="spareVis"
      @ok="spareHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="姓名">
                <a-input v-model="queryParam.condition"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="执行班组">
                <j-dict-select-tag type="list" v-model="queryParam.spareType" dictCode="spare_type"></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <!-- <j-dict-select-tag
                  type="list"
                  v-model="queryParam.storeHouse"
                  dictCode="spare_type"
                ></j-dict-select-tag> -->
                <a-tree-select
                  v-model.trim="queryParam.storeHouse"
                  :treeData="treeData"
                  style="width: 100%"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  :tree-data="treeData"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="spareTable.columns"
        :dataSource="spareTable.dataSource"
        :pagination="spareTable.ipagination"
        :loading="spareTable.loading"
        :rowSelection="{ selectedRowKeys: spareTableSelectedRowKeys, onChange: spareTableOnSelectChange }"
        class="j-table-force-nowrap"
        @change="spareTableHandleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      width="60%"
      :visible="equipmentVis"
      @ok="handleOk"
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="equipmentParam.condition"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类型">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="equipmentParam.spareType"
                  dictCode="spare_type"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="equipmentParam.field"
                  :treeData="treeData"
                  style="width: 100%"
                  :tree-data="treeData"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  placeholder="请选择使用部门"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="equipmentTable.columns"
        :dataSource="equipmentTable.dataSource"
        :pagination="equipmentTable.ipagination"
        :loading="equipmentTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio' }"
        class="j-table-force-nowrap"
        @change="handleEquipmentTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { axios } from '@/utils/request'
import { loadCategoryData, queryDepartTreeList, getUserList } from '@/api/api'
import JUpload from '@/components/jeecg/JUpload'
import { filterObj } from '@/utils/util'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

export default {
  name: 'TabTroubleFixReportForm',
  components: {
    JUpload
  },
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
      fileList: [],
      // 添加人员
      spareVis: false,
      editDisabled: false,
      executeT: false, //执行外委
      verifyT: false, //验证
      spareTable: {
        loading: false,
        dataSource: [],
        ipagination: {
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
            title: '名称',
            align: 'center',
            dataIndex: 'realname'
          },
          {
            title: '性别',
            align: 'center',
            dataIndex: 'sex_dictText'
          },
          {
            title: '职务',
            align: 'center',
            dataIndex: 'post'
          },
          {
            title: '部门',
            align: 'center',
            dataIndex: 'orgCodeTxt'
          },
          {
            title: '执行班组',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '80px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '姓名',
            key: 'realname',
            width: '3%',
            type: FormTypes.normal
          },
          {
            title: '维修开始时间',
            key: 'count',
            width: '10%',
            type: FormTypes.datetime,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '维修完成时间',
            key: 'unitPrice',
            width: '10%',
            type: FormTypes.datetime,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '维修用时(小时)',
            key: 'amount',
            width: '10%',
            type: FormTypes.normal,
            validateRules: [{ required: false, message: '可修改单位为分钟数' }]
          },
          {
            title: '实际维修工时',
            key: 'unitPrice',
            width: '10%',
            type: FormTypes.input,
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '内容或步骤',
            key: 'ddd',
            width: '10%',
            type: FormTypes.input
          }
        ]
      },
      // end
      // 选择设备
      treeData: [],
      dictOptions: {},
      disableSubmit: false,
      loading: false,
      dataSource: [],
      selectedRowKeys: [],
      selectionRows: [],
      // 添加人员
      spareTableSelectedRowKeys: [],
      spareTableSelectionRows: [],
      spareColumns: [
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
          // dataIndex: 'sysOrgCode'
          dataIndex: 'depart'
        },
        {
          title: '设备类型',
          align: 'center',
          dataIndex: 'equipmentType',
          customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
        }
      ],
      equipmentTable: {
        loading: false,
        dataSource: [],
        ipagination: {
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
      queryParam: {
        documentType: '7'
      },
      equipmentParam: {},
      equipmentVis: false,
      ipagination: {
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
      // end
      model: {
        fixWorkOrderWorkloads: []
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {
        entrustTel: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      url: {
        add: '/fixworkorder/outFixEntrust/add',
        edit: '/fixworkorder/outFixEntrust/edit',
        executeOutsourcing: '/fixworkorder/outFixEntrust/executeOutsourcing', //执行外委 post
        verification: '/fixworkorder/outFixEntrust/verification', //外委验证 post
        queryById: '/fixworkorder/outFixEntrust/queryById',
        equipmentList: '/equipment/equipment/list' //设备管理分页
      }
    }
  },
  computed: {
    // end
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.initDictConfig()
    getAction('/equipment/equipment/newTroubleList').then(res => {
      if (res.success) {
        this.dataSource = res.result.records || []
        console.log('  this.dataSource', this.dataSource)
      }
    })

    // 用户信息
    getUserList().then(res => {
      if (res.success) {
        this.spareTable.dataSource = res.result.records || []
      }
    })
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    validatePhone(rule, value, callback) {
      if (value) {
        var myreg = /^[1][3,4,5,7,8][0-9]{9}$/
        if (!myreg.test(value)) {
          callback('请输入正确的手机号')
        } else {
          callback()
        }
      } else {
        callback()
      }
    },
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.equipmentTable.ipagination = pagination
      this.equipmentLoad()
    },
    equipmentLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.equipmentTable.ipagination.current = 1
      }
      var params = Object.assign(this.equipmentParam)
      params.model = 0
      params.pageNo = this.equipmentTable.ipagination.current
      params.pageSize = this.equipmentTable.ipagination.pageSize
      this.equipmentTable.loading = true
      getAction(this.url.equipmentList, params, this.isorter)
        .then(res => {
          if (res.success) {
            this.equipmentTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.equipmentTable.ipagination.total = res.result.total
            } else {
              this.equipmentTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.equipmentTable.loading = false
        })
    },
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'

        if (sum > 30) {
          this.$message.warning('文件总大小超过30MB，请修改后重新上传!')
        }
      }
      return false
    },
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          console.log('选择设备.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    spareTableOnSelectChange(spareTableSelectedRowKeys, spareTableSelectionRows) {
      this.spareTableSelectedRowKeys = spareTableSelectedRowKeys
      this.spareTableSelectionRows = spareTableSelectionRows
    },
    spareTableHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.spareTable.ipagination = pagination
      this.loadData()
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.loadData()
    },
    selectEquipment() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    handleOk() {
      this.model.equipmentCode = this.selectionRows[0].equipmentCode
      this.model.equipmentName = this.selectionRows[0].equipmentName
      this.model.equipmentType = this.selectionRows[0].equipmentType
      this.model.equipmentModel = this.selectionRows[0].equipmentModel
      this.model.equipmentDepart = this.selectionRows[0].sysOrgCode
      // console.log('this.model', this.model)
      // console.log('this.selectionRows', this.selectionRows)
      this.equipmentVis = false
    },
    spareHandleOk() {
      this.table.dataSource = this.spareTableSelectionRows
      this.spareVis = false
    },
    handleCancel() {
      this.equipmentVis = false
      this.spareVis = false
    },
    searchQuery() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    searchReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction('/equipment/equipment/newTroubleList', params)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
    },
    // end
    // 关联图片
    /** upload headers */
    uploadHeaders() {
      let { originColumn: col } = this
      let headers = {}
      if (col.token === true) {
        headers['X-Access-Token'] = this.$ls.get(ACCESS_TOKEN)
      }
      return headers
    },
    /** 上传请求地址 */
    uploadAction() {
      // if (!this.originColumn.action) {
      return window._CONFIG['domianURL'] + '/sys/common/upload'
      // } else {
      //   return this.originColumn.action
      // }
    },
    handleChangeUpload(info) {
      let { originColumn: col } = this
      let { file } = info
      let value = {
        name: file.name,
        type: file.type,
        size: file.size,
        status: file.status,
        percent: file.percent
      }
      if (file.response) {
        value['responseName'] = file.response[this.responseName]
      }
      if (file.status === 'done') {
        if (typeof file.response.success === 'boolean') {
          if (file.response.success) {
            value['path'] = file.response[this.responseName]
            this.handleChangeCommon(value)
          } else {
            value['status'] = 'error'
            value['message'] = file.response.message || '未知错误'
          }
        } else {
          // 考虑到如果设置action上传路径为非jeecg-boot后台，可能不会返回 success 属性的情况，就默认为成功
          value['path'] = file.response[this.responseName]
          this.handleChangeCommon(value)
        }
      } else if (file.status === 'error') {
        value['message'] = file.response.message || '未知错误'
      }
      this.innerFile = value
    },
    // end

    uploadFilesToServer(uploadApiUrl, fileName, files) {
      let formData = new FormData()
      formData.append(fileName, files)
      //添加请求头
      const headers = {
        'Content-Type': 'multipart/form-data'
      }
      //配置头
      const request = axios.create({
        headers: headers
      })
      //开始上传
      return request
        .post(uploadApiUrl, formData)
        .then(response => {
          return Promise.resolve(response.data)
        })
        .catch(error => {
          return Promise.reject(error)
        })
    },

    selectSparePart() {
      this.spareVis = true
    },
    delSparePart() {
      console.log('删除')
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.editDisabled = true
    },
    execute(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.editDisabled = true
      this.executeT = true
    },
    verify(record) {
      this.model = Object.assign({}, record)
      this.model.repair = '0'
      this.visible = true
      this.editDisabled = true
      this.verifyT = true
    },
    submitForm() {
      const that = this
      // this.model.fixWorkOrderWorkloads = this.table.dataSource
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            if (that.verifyT) {
              httpurl += that.url.verification
              method = 'post'
            } else if (that.executeT) {
              httpurl += that.url.executeOutsourcing
              method = 'post'
            } else {
              httpurl += that.url.edit
              method = 'put'
            }
          }
          httpAction(httpurl, this.model, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
              that.executeT = false
              that.verifyT = false
            })
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
.ant-modal-body {
  margin-bottom: 0 !important;
}
</style>
