<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model v-if="!disExecute" ref="form" :model="model" :rules="validatorRules">
      <!-- 设备信息 -->
      <a-row style="margin-top:20px">
        <a-button :disabled="formDisabled" v-if="!formDisabled" @click="selectSparePart" class="new-btn" type="primary">
          <a-icon type="plus" />
          选择设备
        </a-button>
        <j-vxe-table
          ref="editableTable1"
          row-number
          :rowSelection="false"
          keep-source
          :height="300"
          :loading="table.loading"
          :dataSource="table.dataSource"
          :columns="table.columns"
          :disabled="disEdit"
          style="margin-top: 8px;"
        />
      </a-row>
      <!-- 新增、编辑、查看 -->
      <a-row style="margin-top:20px">
        <a-col :span="24"><div class="title">检验信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="检验方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkWay">
            <a-radio-group :disabled="formDisabled" v-model="model.checkWay">
              <a-radio value="1">内检</a-radio>
              <a-radio value="2">外检</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkType">
            <a-select
              :showSearch="true"
              optionFilterProp="label"
              :disabled="formDisabled"
              v-model.trim="model.checkType"
              style="width: 100%"
              placeholder="请选择"
            >
              <a-select-option value="1" label="定期检验">定期检验</a-select-option>
              <a-select-option value="2" label="监督检验">监督检验</a-select-option>
            </a-select>
            <!-- <j-dict-select-tag
              :disabled="formDisabled"
              type="list"
              dictCode="checkPlanType"
              v-model.trim="model.checkType"
              placeholder="请选择检验类型"
            ></j-dict-select-tag> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="循环周期" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input :disabled="formDisabled" v-model.trim="model.cycleTime" style="width: 60%;margin-right: 10px;">
            </a-input>
            <a-select :disabled="formDisabled" v-model.trim="model.circleWay" style="width: 35%">
              <a-select-option v-for="(item, index) in cycleOption" :key="index" :value="item.value">
                {{ item.label }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="下次检验时间"
            :rules="[{ required: true, message: '请选择下次检验时间!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="nextCheckTime"
          >
            <j-date
              :disabled="formDisabled"
              placeholder="请选择下次检验时间"
              v-model="model.nextCheckTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input :disabled="formDisabled" v-model="model.remark" placeholder="请输入备注"></a-input>
          </a-form-model-item>
        </a-col>
        <!-- end -->
      </a-row>
    </a-form-model>
    <!-- 执行检验计划 -->
    <a-form-model ref="formRef" :model="formData" :rules="formRules" v-if="disExecute">
      <a-row style="margin-top:20px">
        <a-col :span="24"><div class="title">执行检验计划</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="检验方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkWay">
            <a-radio-group :disabled="true" v-model="formData.checkWay">
              <a-radio value="1">内检</a-radio>
              <a-radio value="2">外检</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkType">
            <a-select v-model.trim="formData.checkType" style="width: 100%" placeholder="请选择">
              <a-select-option value="">全部</a-select-option>
              <a-select-option value="1">定期检验</a-select-option>
              <a-select-option value="2">监督检验</a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="送检时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="submissionTime">
            <j-date
              :disabled="formDisabled"
              placeholder="请选择送检时间"
              v-model="formData.submissionTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="送检人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="submissionPerson">
            <a-input :disabled="formDisabled" v-model="formData.submissionPerson" placeholder="请输入送检人"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkStatus">
            <a-radio-group :disabled="formDisabled" v-model="formData.checkStatus">
              <a-radio value="1">在检</a-radio>
              <a-radio value="2">结束</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkoutPerson">
            <a-input :disabled="formDisabled" v-model="formData.checkoutPerson" placeholder="请输入检验员"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="检验单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkoutUnit">
            <!-- <a-input :disabled="formDisabled" v-model="formData.checkoutUnit" placeholder="请选择检验单位"></a-input> -->
            <a-select
              show-search
              :disabled="formDisabled"
              :filterOption="filterOption"
              v-model="formData.checkoutUnit"
              placeholder="回车可搜索"
              style="width: 100%"
            >
              <a-select-option v-for="(item, index) in equipmentSupplierOptions" :key="index" :value="item.id">
                {{ item.companyName }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="phone">
            <a-input :disabled="formDisabled" v-model="formData.phone" placeholder="请输入电话"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input :disabled="formDisabled" v-model="formData.remark" placeholder="请输入备注"></a-input>
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-row>
        <j-vxe-table
          :scroll="{ x: true }"
          bordered
          rowKey="id"
          :columns="planTable.columns"
          :dataSource="planTable.dataSource"
          class="j-table-force-nowrap"
        >
          <template slot="htmlSlot" slot-scope="text">
            <div v-html="text"></div>
          </template>
        </j-vxe-table>
      </a-row>
    </a-form-model>
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
                <a-input placeholder="编码，名称，型号" v-model.trim="queryParam.dd"></a-input>
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
import { httpAction, getAction, postAction } from '@/api/manage'
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
      formData: {},
      equipmentSupplierOptions: [],
      planId: '',
      planTable: {
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
      disEdit: false,
      disExecute: false,
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '设备编号',
            key: 'equipmentCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'equipmentName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'equipmentModel',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '上次检验时间',
            // key: 'equipmentModel',
            width: '10%',
            type: FormTypes.normal
          }
        ]
      },
      // 选择设备
      queryParam: {},
      width: 900,
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
        checkWay: 1,
        // checkStatus: 0,
        circleWay: 'time1'
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 8 }
      },
      confirmLoading: false,
      validatorRules: {
        phone: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      formRules: {
        phone: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }],
        submissionTime: { required: true, message: '请选择送检时间!' }
      },
      url: {
        add: '/checkout/tabCheckoutPlan/add',
        edit: '/checkout/tabCheckoutPlan/edit',
        queryById: '/checkout/tabCheckoutPlan/queryById',
        ExecuteCheckOutPlanInfo: '/checkout/tabCheckoutPlan/ExecuteCheckOutPlanInfo', //执行检验
        equipmentList: '/equipment/equipment/list' //设备管理分页
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //生产商、供应商
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        // console.log('res.result.records', res.result.records)
        // console.log('res.供应商', res.result)
        this.equipmentSupplierOptions = arr
      }
    })
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
    filterOption(input, option) {
      return option.componentOptions.children[0].children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
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
      var data = []
      // this.table.dataSource = this.selectionRows
      this.selectionRows.forEach(item => {
        let pm = {
          deviceId: item.id,
          equipmentCode: item.equipmentCode,
          equipmentName: item.equipmentName,
          equipmentModel: item.equipmentModel
        }
        this.table.dataSource.push(pm)
      })
      this.spareVis = false
      console.log('this.selectionRows', this.selectionRows)
      console.log('  this.table.dataSource', this.table.dataSource)
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleCancel() {
      this.spareVis = false
    },
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
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
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
    // handleDevice() {
    //   this.$refs.ChoiceDevice.selectSparePart()
    // },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
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
        this.disEdit = false
      }
      this.visible = true
    },
    execute(record) {
      console.log('record', record)
      if (record) {
        record.forEach(item => {
          let pm = {
            deviceCode: item.deviceCode,
            deviceCode: item.deviceCode,
            deviceName: item.deviceName,
            deviceModel: item.deviceModel,
            deviceId: item.deviceId
          }
          this.planTable.dataSource.push(pm)
        })
        this.$set(this.formData, 'checkStatus', '1')
        var checkWay
        if (record[0].checkWay == '内检') {
          checkWay = '1'
        }
        if (record[0].checkWay == '外检') {
          checkWay = '2'
        }
        this.formData.checkWay = checkWay
        this.planId = record[0].id
        console.log('this.planId', this.planId)
      }
      this.visible = true
      this.disExecute = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      if (that.disExecute) {
        that.$refs.formRef.validate(valid => {
          if (valid) {
            this.planTable.dataSource.map(item => delete item.id)
            let pm = {
              pd: this.planTable.dataSource,
              checkRecords: this.formData,
              planId: this.planId
            }
            postAction(this.url.ExecuteCheckOutPlanInfo, pm)
              .then(res => {
                if (res.success) {
                  that.$message.success(res.message)
                  that.$emit('ok')
                } else {
                  that.$message.warning(res.message)
                }
              })
              .finally(() => {
                that.model = {}
                that.confirmLoading = false
              })
          }
        })
      } else {
        that.$refs.form.validate(valid => {
          if (valid) {
            if (this.table.dataSource.length != 0) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''
              if (!this.model.id) {
                httpurl += this.url.add
                method = 'post'
                // 拼接deviceIds
                that.model.deviceIds = ''
                console.log('that.table.dataSource', that.table.dataSource)
                that.table.dataSource.forEach(val => {
                  that.model.deviceIds += val.deviceId + ','
                })
                that.model.deviceIds =
                  that.model.deviceIds.substring(that.model.deviceIds.length - 1) == ','
                    ? that.model.deviceIds.substring(0, that.model.deviceIds.length - 1)
                    : that.model.deviceIds
                // end
              } else {
                httpurl += this.url.edit
                method = 'put'
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
                  that.model = {}
                  that.confirmLoading = false
                })
            } else {
              that.$message.warning('至少选择一个设备！')
            }
          }
        })
      }
    }
  }
}
</script>
