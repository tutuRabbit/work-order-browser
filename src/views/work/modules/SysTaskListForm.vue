<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="工单编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="projectCode">
              <a-input disabled v-model="model.projectCode" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="产品编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productCode">
              <a-input disabled v-model="model.productCode" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName">
              <a-input disabled v-model="model.productName" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="工序编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processCode">
              <a-input disabled v-model="model.procedureCode" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="工序名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processName">
              <a-input disabled v-model="model.procedureName" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="计划开始时间"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="procedureStartTime"
            >
              <j-date
                placeholder="请选择计划开始时间"
                v-model="model.procedureStartTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="计划结束时间"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="procedureEndTime"
            >
              <j-date
                placeholder="请选择计划结束时间"
                v-model="model.procedureEndTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="报工权限" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputObject">
              <a-input disabled v-model="model.outputObject" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="分配列表" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskOperators">
              <a-select
                style="width: 100%"
                mode="multiple"
                placeholder="请选择"
                v-model="model.taskOperators"
                :allowClear="true"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option
                  v-for="(item, index) in distributionList"
                  :key="index"
                  :value="item.value"
                  :label="item.title"
                  :disabled="item.disabled"
                  >{{ item.title }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="生产车间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputRate">
              <a-input disabled v-model="model.manufacturingShopStr" placeholder="-"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="报工配比" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputRate">
              <a-input v-model="model.outputRate" placeholder="请输入报工配比"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="计划数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planAmount">
              <a-input-number v-model="model.planAmount" placeholder="请输入计划数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="不良品数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectsAmount">
              <a-input-number disabled v-model="model.defectsAmount" placeholder="-" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="良品数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fineAmount">
              <a-input-number disabled v-model="model.fineAmount" placeholder="-" style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'

export default {
  name: 'SysTaskListForm',
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
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 17 }
      },
      confirmLoading: false,
      validatorRules: {
        procedureCode: [{ required: true, message: '请填写工序编号!' }],
        procedureName: [{ required: true, message: '请填写工序名称!' }],
        procedureStartTime: [{ required: true, message: '请选择计划开始时间!' }],
        procedureEndTime: [{ required: true, message: '请选择计划结束时间!' }],
        outputObject: [{ required: true, message: '请选择报工权限!' }],
        outputRate: [{ required: true, message: '请输入报工配比!' }],
        planAmount: [{ required: true, message: '请输入计划数!' }]
      },
      url: {
        add: '/longxianda/sysTaskList/add',
        edit: '/longxianda/sysTaskList/edit',
        queryById: '/longxianda/sysTaskList/queryById?id='
      },
      distributionList: [] // 分配列表
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
  },
  methods: {
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      if (this.model.id) {
        // getAction(this.url.queryById + record.id).then(res => {
        //   this.model = res.result
        // })
        if (this.model.taskOperators) {
          this.model.taskOperators = this.model.taskOperators.split(',')
        } else {
          this.model.taskOperators = []
        }
        this.getDistributionList(this.model.procedureId)
      }
    },
    getDistributionList(procedureId) {
      var that = this
      let alist = []
      let httpurl = '/longxianda/sysProcedure/getUserByProcedureId?procedureId=' + procedureId
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.length > 0) {
            res.map(item => {
              alist.push({
                title: item.realname,
                value: item.id
              })
            })
            if (this.model.procedureStatus !== 0) {
              console.log('this.model.taskOperators', this.model.taskOperators)
              let arr = this.model.taskOperators
              alist.map(item => {
                item.disabled = false
                arr.map(val => {
                  if (val == item.value) {
                    item.disabled = true
                  }
                })
              })
            }
            that.distributionList = alist
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    seveEistribution(cellValue) {
      var that = this
      let httpurl = '/longxianda/sysTaskList/edit'
      httpAction(httpurl, { id: that.model.id, taskOperators: cellValue }, 'post')
        .then(res => {
          if (res.code == 200) {
            // that.$message.success('保存成功！')
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
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
            })
          if (this.model.taskOperators) {
            this.model.taskOperators = this.model.taskOperators.join(',')
            this.seveEistribution(this.model.taskOperators)
          }
        }
      })
    }
  }
}
</script>
