<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="6">
            <a-form-item label="年度保养计划" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag
                type="list"
                dictCode="year_plan_yearth"
                v-model.trim="model.keepYearth"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row>
          <a-button :disabled="formDisabled" class="new-btn" @click="selectSparePart" type="primary">
            <a-icon type="plus" />
            选择设备
          </a-button>
          <!-- <a-button :disabled="formDisabled" type="primary">
            <a-icon type="plus" />
            批设保养级别
          </a-button>
          <a-button :disabled="formDisabled" type="primary">
            <a-icon type="plus" />
            批设保养标准
          </a-button> -->
          <a-button :disabled="formDisabled" type="primary">
            <a-icon type="plus" />
            删除
          </a-button>
          <j-vxe-table
            ref="personnel"
            row-number
            row-selection
            keep-source
            :height="500"
            :loading="personnel.loading"
            :dataSource="personnel.dataSource"
            :columns="personnel.columns"
            style="margin-top: 8px;"
          >
            <template v-slot:action="props">
              <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
                <a>删除</a>
              </a-popconfirm>
            </template>
          </j-vxe-table>
        </a-row>
      </a-form-model>
      <!-- 选择设备 -->
      <j-modal
        title="选择设备"
        width="60%"
        :visible="equipmentVis"
        @ok="handleOk"
        @cancel="handleCancel"
        cancelText="关闭"
      >
        <div class="title" style="margin-bottom:20px">基本信息</div>
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline" @keyup.enter.native="searchEquipment">
            <a-row :gutter="24">
              <a-col :span="8">
                <a-form-item label="搜索条件">
                  <a-input placeholder="编码，名称，型号" v-model.trim="equipmentParam.search"></a-input>
                </a-form-item>
              </a-col>

              <a-col :span="5">
                <a-form-item label="设备类别">
                  <j-category-select v-model="equipmentParam.equipmentType" pcode="B03" />
                </a-form-item>
              </a-col>
              <a-col :span="5">
                <a-form-item label="使用部门">
                  <a-tree-select
                    v-model.trim="equipmentParam.sysOrgCode"
                    style="width: 100%"
                    :tree-data="treeData"
                    placeholder="请选择使用部门"
                  >
                  </a-tree-select>
                </a-form-item>
              </a-col>
              <a-col :span="4">
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <a-button type="primary" @click="searchEquipment" icon="search">查询</a-button>
                  <a-button type="primary" @click="searchEquipmentReset" icon="reload" style="margin-left: 8px"
                    >重置</a-button
                  >
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
          :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
          class="j-table-force-nowrap"
          @change="handleEquipmentTableChange"
        >
          <template slot="htmlSlot" slot-scope="text">
            <div v-html="text"></div>
          </template>
        </a-table>
      </j-modal>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { queryDepartTreeList } from '@/api/api'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'TabKeepYearPlanForm',
  components: {},
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
      model: {},
      equipmentVis: false,
      equipmentParam: {},
      selectedRowKeys: [],
      selectionRows: [],
      treeData: [],
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
      personnel: {
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
            title: '设备id',
            width: '1%',
            key: 'deviceId',
            type: FormTypes.normal
          },
          {
            title: '设备编码',
            width: '8%',
            key: 'equipmentCode',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'equipmentName',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'equipmentModel',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '设备类型',
            key: 'equipmentType',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '使用部门',
            key: 'depart',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '保养级别',
            key: 'keepLevel',
            type: FormTypes.select,
            width: '8%',
            options: [],
            dictCode: 'keep_level',
            placeholder: '请选择${title}'
          },
          {
            title: '保养标准',
            key: 'keepStandard',
            width: '8%',
            type: FormTypes.select,
            options: [],
            dictCode: 'keep_level',
            placeholder: '请选择${title}'
          },
          {
            title: '保养时间',
            width: '8%',
            key: 'keepTime',
            type: FormTypes.select,
            options: [],
            dictCode: 'keep_plan_times'
          },
          {
            title: '1月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month1'
          },
          {
            title: '2月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month2'
          },
          {
            title: '3月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month3'
          },
          {
            title: '4月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month4'
          },
          {
            title: '5月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month5'
          },
          {
            title: '6月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month6'
          },
          {
            title: '7月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month7'
          },
          {
            title: '8月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month8'
          },
          {
            title: '9月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month9'
          },
          {
            title: '10月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month10'
          },
          {
            title: '11月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month11'
          },
          {
            title: '12月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month12'
          }
        ]
      },
      dictOptions: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {
        keepYearth: [{ required: true, message: '请选择年度保养计划!' }]
      },
      url: {
        add: '/year/tabKeepYearPlan/add',
        edit: '/year/tabKeepYearPlan/edit',
        equipmentList: '/equipment/equipment/list', //设备管理分页
        queryById: '/year/tabKeepYearPlan/queryById'
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    // 设备管理
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleOk() {
      this.personnel.dataSource = this.selectionRows
      this.equipmentVis = false
    },
    handleCancel() {
      this.equipmentVis = false
    },
    searchEquipment() {},
    searchEquipmentReset() {},
    selectSparePart() {
      this.equipmentLoad()
      this.equipmentVis = true
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
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      let pm = {
        yearPlans: this.personnel.dataSource,
        keepYearth: this.model.keepYearth
      }
      console.log('that.model.keepYearth', that.model.keepYearth)
      if (that.model.keepYearth == '' || that.model.keepYearth == undefined) {
        that.$message.warning('请选择年度保养计划！')
      } else if (that.personnel.dataSource.length == 0) {
        that.$message.warning('请选择设备添加计划！！')
      } else {
        that.confirmLoading = true
        let httpurl = ''
        let method = ''
        if (!that.model.id) {
          httpurl += that.url.add
          method = 'post'
        } else {
          httpurl += that.url.edit
          method = 'put'
        }
        httpAction(httpurl, pm, method)
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
          })
      }
    }
  }
}
</script>
<style>
.ant-btn-primary {
  margin-left: 10px;
}
</style>
