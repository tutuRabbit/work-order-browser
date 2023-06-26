<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <a-row style="margin-top:20px">
        <a-col :span="24"><div class="title">检验信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="检验方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkWay">
            <a-radio-group v-model="model.checkWay" :disabled="formDisabled">
              <a-radio value="1">内检</a-radio>
              <a-radio value="2">外检</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkType">
            <a-select
              :disabled="formDisabled"
              :showSearch="true"
              optionFilterProp="label"
              v-model.trim="model.checkType"
              style="width: 100%"
              placeholder="请选择检验类型"
            >
              <a-select-option value="1" label="定期检验">定期检验</a-select-option>
              <a-select-option value="2" label="监督检验">监督检验</a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkStatus">
            <a-radio-group :disabled="formDisabled" v-model="model.checkStatus">
              <a-radio value="1">在检</a-radio>
              <a-radio value="2">结束</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkoutPlanRecordCode">
            <a-input
              :disabled="formDisabled"
              v-model="model.checkoutPlanRecordCode"
              placeholder="不填可自动生成"
            ></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="送检时间"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="submissionTime"
            :rules="[{ required: true, message: '请选择送检时间!' }]"
          >
            <j-date
              :disabled="formDisabled"
              placeholder="请选择送检时间"
              v-model="model.submissionTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="送检人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="submissionPerson">
            <a-input :disabled="formDisabled" v-model="model.submissionPerson" placeholder="请输入送检人"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkoutPerson">
            <a-input :disabled="formDisabled" v-model="model.checkoutPerson" placeholder="请输入检验员"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="送检单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkoutUnit">
            <a-input :disabled="formDisabled" v-model="model.checkoutUnit" placeholder="请输入送检单位"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="phone">
            <a-input :disabled="formDisabled" v-model="model.phone" placeholder="请输入电话"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="备注" :labelCol="labelA" :wrapperCol="wrapperA" prop="remark">
            <a-textarea :disabled="formDisabled" rows="4" v-model="model.remark" placeholder="请输入备注" />
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <a-button v-if="!formDisabled" @click="selectPlan" class="new-btn" type="primary">
          <a-icon type="plus" />
          选择计划
        </a-button>
        <a-button v-if="!formDisabled" style="margin-left:20px" @click="selectSparePart" class="new-btn" type="primary">
          <a-icon type="plus" />
          选择设备
        </a-button>
        <j-vxe-table
          ref="editableTable1"
          row-number
          row-selection
          keep-source
          :height="300"
          :loading="table.loading"
          :dataSource="table.dataSource"
          :columns="table.columns"
          :disabled="disEdit ? disEdit : formDisabled"
          style="margin-top: 8px;"
        />
      </a-row>
    </a-form-model>
    <!-- 选择计划 -->
    <j-modal
      title="选择计划"
      :width="width"
      :visible="planVis"
      @ok="planHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">选择计划</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="planQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="planParam.search"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类别">
                <j-category-select v-model="planParam.equipmentType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="planParam.sysOrgCode"
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
                <a-button type="primary" @click="planQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="planReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        :bordered="false"
        rowKey="id"
        :columns="planTable.columns"
        :dataSource="planTable.dataSource"
        :pagination="planTable.ipagination"
        :loading="planTable.loading"
        :rowSelection="{ selectedRowKeys: planRowKeys, onChange: planOnSelectChange }"
        class="j-table-force-nowrap"
        @change="planHandleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      :width="width"
      :visible="spareVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">选择设备</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="queryParam.search"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.equipmentType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="queryParam.sysOrgCode"
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
        :bordered="false"
        rowKey="id"
        :columns="deviceTable.columns"
        :dataSource="deviceTable.dataSource"
        :pagination="deviceTable.ipagination"
        :loading="deviceTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleDeviceTableChange"
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
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { loadCategoryData, queryDepartTreeList } from '@/api/api'
export default {
  name: 'TabCheckoutPlanForm',
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
      disEdit: false,
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '计划编号',
            key: 'planCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '设备编号',
            key: 'deviceCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'deviceName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'deviceModel',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '检验结论',
            key: 'checkResult',
            width: '10%',
            type: FormTypes.select,
            options: [
              { title: '合格', value: '1' },
              { title: '报废', value: '2' },
              { title: '维修', value: '3' }
            ]
          },
          {
            title: '证书编号',
            key: 'certificateNumber',
            width: '10%',
            type: FormTypes.input
          },
          {
            title: '费用',
            key: 'money',
            width: '10%',
            type: FormTypes.inputNumber
          }
        ]
      },
      // 选择计划
      planVis: false,
      planParam: {},
      // 选择设备
      queryParam: {},
      width: '80%',
      spareVis: false,
      cycleOption: [
        {
          label: '天',
          value: 'time1'
        },
        {
          label: '月',
          value: 'time2'
        },
        {
          label: '季',
          value: 'time3'
        }
      ],
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
      planTable: {
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
            dataIndex: 'deviceCode'
          },
          {
            title: '设备名称',
            align: 'center',
            dataIndex: 'deviceName'
          },
          {
            title: '规格型号',
            align: 'center',
            dataIndex: 'deviceModel'
          },
          {
            title: '下次检验时间',
            align: 'center',
            dataIndex: 'nextCheckTime'
          },
          {
            title: '检验方式',
            align: 'center',
            dataIndex: 'checkWay'
          },
          {
            title: '检验类型',
            align: 'center',
            dataIndex: 'checkType'
          }
          // {
          //   title: '使用部门',
          //   align: 'center',
          //   dataIndex: 'depart'
          // },
          // {
          //   title: '设备类型',
          //   align: 'center',
          //   dataIndex: 'equipmentType',
          //   customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          // }
        ]
      },
      planRowKeys: [],
      planRows: [],
      deviceTable: {
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
      treeData: [],
      dictOptions: {},
      selectedRowKeys: [],
      model: {
        checkWay: '1',
        checkType: '1',
        checkStatus: '1'
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 8 }
      },
      labelA: {
        xs: { span: 24 },
        sm: { span: 2 }
      },
      wrapperA: {
        xs: { span: 24 },
        sm: { span: 20 }
      },
      confirmLoading: false,
      validatorRules: {
        phone: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      url: {
        add: '/check_plan_record/tabCheckoutPlanRecords/add',
        edit: '/check_plan_record/tabCheckoutPlanRecords/edit',
        queryById: '/check_plan_record/tabCheckoutPlanRecords/queryById',
        equipmentList: '/equipment/equipment/list', //设备管理分页
        planList: '/checkout/tabCheckoutPlan/list' //检验计划分页
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
    this.initDictConfig()
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    removeDuplicate() {
      let arr = this.table.dataSource
      this.table.dataSource ==
        arr.reduce(function(item, next) {
          obj[next.key] ? '' : (obj[next.key] = true && item.push(next))
          return item
        }, [])

      this.table.dataSource.forEach(item => {
        delete item.id
        // console.log('dfsdfsdfsdfsdfsdfsd', item.deviceId)
      })
      // console.log(' this.table.dataSource', this.table.dataSource)
    },
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
    onChangeRangeDate(value, dateString) {
      this.model.startTime = dateString[0]
      this.model.endTime = dateString[1]
    },
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    searchReset() {
      this.queryParam = {}
      this.equipmentLoad(1)
    },
    handleOk() {
      this.selectionRows.forEach(item => {
        let pm = {
          deviceId: item.id,
          deviceCode: item.equipmentCode,
          deviceName: item.equipmentName,
          deviceModel: item.equipmentModel
        }
        this.table.dataSource.push(pm)
      })
      // this.table.dataSource = this.removeDuplicate(this.table.dataSource)
      console.log(' this.table.dataSource', this.table.dataSource)
      this.handleCancel()
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleCancel() {
      this.planVis = false
      this.spareVis = false
      this.queryParam = {}
      this.planParam = {}
    },
    // 选择计划
    planQuery() {
      this.planLoad(1)
      // 点击查询清空列表选中行
      this.planRowKeys = []
      this.planRows = []
    },
    planReset() {
      this.planParam = {}
      this.planLoad(1)
    },
    planHandleOk() {
      this.planRows.forEach(item => {
        let pm = {
          planCode: item.id,
          deviceId: item.deviceId,
          deviceCode: item.deviceCode,
          deviceName: item.deviceName,
          deviceModel: item.deviceModel
        }
        this.table.dataSource.push(pm)
      })
      // 去重
      // this.removeDuplicate()
      console.log(' this.table.dataSource', this.table.dataSource)
      this.handleCancel()
    },
    planHandleTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      this.planTable.ipagination = pagination
      this.planLoad()
    },
    planOnSelectChange(planRowKeys, planRows) {
      this.planRowKeys = planRowKeys
      this.planRows = planRows
    },
    planLoad(arg) {
      if (!this.url.planList) {
        this.$message.error('请设置url.planList属性!')
        return
      }
      if (arg === 1) {
        this.planTable.ipagination.current = 1
      }
      var params = Object.assign(this.queryParam)
      params.model = 0
      params.pageNo = this.planTable.ipagination.current
      params.pageSize = this.planTable.ipagination.pageSize
      this.planTable.loading = true
      getAction(this.url.planList, params)
        .then(res => {
          if (res.success) {
            this.planTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.planTable.ipagination.total = res.result.total
            } else {
              this.planTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.planTable.loading = false
        })
    },
    selectPlan() {
      this.planLoad()
      this.planVis = true
    },
    // end
    searchQuery() {
      this.equipmentLoad(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    selectSparePart() {
      this.equipmentLoad()
      this.spareVis = true
    },
    handleDeviceTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      this.deviceTable.ipagination = pagination
      this.equipmentLoad()
    },
    equipmentLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.deviceTable.ipagination.current = 1
      }
      var params = Object.assign(this.queryParam)
      params.model = 0
      params.pageNo = this.deviceTable.ipagination.current
      params.pageSize = this.deviceTable.ipagination.pageSize
      this.deviceTable.loading = true
      getAction(this.url.equipmentList, params, this.isorter)
        .then(res => {
          if (res.success) {
            this.deviceTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.deviceTable.ipagination.total = res.result.total
            } else {
              this.deviceTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.deviceTable.loading = false
        })
    },
    add() {
      // this.edit(this.modelDefault)
      this.disEdit = false
      this.visible = true
    },
    edit(record) {
      // console.log('record', record)
      if (record.equipment) {
        this.disEdit = true
        let pm = {
          deviceId: record.equipment.id,
          equipmentCode: record.equipment.equipmentCode,
          equipmentName: record.equipment.equipmentName,
          equipmentModel: record.equipment.equipmentModel
        }
        this.table.dataSource.push(pm)
        this.model = Object.assign({}, record.checkoutPlan)
      } else {
        getAction('/check_plan_record/tabCheckoutPlanRecords/queryById?id=' + record.id).then(res => {
          this.table.dataSource = res.result.choosePlan
          this.model = Object.assign({}, res.result.record)
          // console.log('res', res)
        })
        this.disEdit = false
      }
      this.visible = true
    },

    submitForm() {
      const that = this
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
            httpurl += this.url.edit
            method = 'put'
          }
          let pm = {
            checkRecords: this.model,
            pd: this.table.dataSource
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
      })
    }
  }
}
</script>
<style scoped>
/* .ant-row {
  display: none;
} */
</style>
