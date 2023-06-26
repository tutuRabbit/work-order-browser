<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item
              label="产品编码"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="produceCode"
              :rules="[{ required: true, message: '请填写产品编码!' }]"
            >
              <a-select
                defaultValue="alipay"
                placeholder="请选择"
                v-model="model.produceCode"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option
                  v-for="(item, index) in produceCodeList"
                  :key="index"
                  @click="fieldsChange(item)"
                  :value="item.code"
                  >{{ item.code }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.produceCode">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="produceName">
              {{ model.produceName }}
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.produceCode">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="produceSpec">
              {{ model.produceSpec }}
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="produceName">
              <a-select defaultValue="alipay" placeholder="请选择" v-model="model.produceName">
                <a-select-option v-for="(item, index) in produceCodeList" :key="index" :value="item.produceName"
                  >{{ item.produceName }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="produceSpec">
              <a-input v-model="model.produceSpec" placeholder="请输入产品规格"></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <a-form-model-item
              label="工序名称"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="processCode"
              :rules="[{ required: true, message: '请填写工序名称!' }]"
            >
              <a-select
                :disabled="ProcedureValid"
                defaultValue="alipay"
                placeholder="请选择"
                v-model="model.processCode"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option
                  v-for="(item, index) in ProcedureList"
                  :key="index"
                  :value="item.code"
                  @click="processChange(item.codeName)"
                  >{{ item.codeName }}
                </a-select-option>
              </a-select>
              <!-- <a-select defaultValue="alipay" placeholder="请选择" v-model="model.processName">
                <a-select-option v-for="(item, index) in ProcedureList" :key="index" :value="item.name"
                  >{{ item.name }}
                </a-select-option>
              </a-select> -->
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="计价方法" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="valuationMothod">
              <!-- <a-select
                defaultValue="alipay"
                placeholder="请选择"
                style="width: width: 100%"
                v-model="model.valuationMothod"
              >
                <a-select-option v-for="(item, index) in valuationList" :key="index" :value="item.value"
                  >{{ item.label }}
                </a-select-option>
              </a-select> -->
              <j-dict-select-tag v-model="model.valuationMothod" placeholder="请选择" dictCode="valuation_type" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="标准产出" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="standardOutput">
              <a-input-number v-model="model.standardOutput" placeholder="请输入标准产出" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="this.model.valuationMothod == 1">
            <a-form-model-item
              label="标准工时(秒)"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="standardTaskTime"
            >
              <a-input-number v-model="model.standardTaskTime" placeholder="请输入标准工时(秒)" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="工资单价"
              :rules="[{ required: true, message: '请输入工资单价!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="unitPrice"
            >
              <!-- <a-input v-model="model.formula" placeholder="请输入工资单价" style="width: 100%" /> -->
              <a-input-number v-model="model.unitPrice" placeholder="请输入工资单价" style="width: 100%" />
            </a-form-model-item>
          </a-col>

          <a-col :span="24">
            <a-form-model-item
              label="工资系数"
              :rules="[{ required: true, message: '请输入工资系数!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="coefficientSalary"
            >
              <a-input-number v-model="model.coefficientSalary" placeholder="请输入工资系数" style="width: 100%" />
            </a-form-model-item>
          </a-col>

          <a-col :span="24">
            <a-form-model-item
              label="标准工资"
              :rules="[{ required: true, message: '请输入标准工资!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="standardSalary"
            >
              <a-input-number
                v-model="model.standardSalary"
                placeholder="请输入标准工资"
                style="width: 100%"
                disabled
              />
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
//标准工资=工资系数*工资单价

export default {
  name: 'SysMeritPayForm',
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
      ProcedureValid: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 4 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 18 }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysMeritPay/add',
        edit: '/longxianda/sysMeritPay/edit',
        queryById: '/longxianda/sysMeritPay/queryById'
      },
      valuationList: [],
      produceCodeList: null,
      ProcedureList: null
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  watch: {
    //coefficientSalary
    'model.coefficientSalary': function(newVal) {
      this.model.standardSalary = newVal * this.model.unitPrice
    },
    'model.unitPrice': function(newVal) {
      this.model.standardSalary = newVal * this.model.coefficientSalary
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.loadProduceCodeList()
    // 工序名称
    // this.loadProcedureList()
    // 获取计价方法
    // this.loadvaluationList()
  },
  methods: {
    processChange(name) {
      console.log('name', name)
      this.model.processName = name
    },
    add() {
      // this.edit(this.modelDefault)
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      getAction(this.url.queryById, { id: record.id }).then(res => {
        console.log('res', res)
        this.model = res.result
        this.ProcedureList = res.result.lineInfos
        // console.log('this.ProcedureList', this.ProcedureList)
      })
      // this.model = Object.assign({}, record)
      this.ProcedureValid = true
      this.visible = true
    },
    fieldsChange(info) {
      console.log('info', info)
      let that = this
      // 产品名称
      that.model.produceName = info.productName
      // 产品规格
      that.model.produceSpec = info.productNorms
      that.loadProcedureList(info.craftRouteId)
    },
    // 获取产品列表
    loadProduceCodeList() {
      var that = this
      let httpurl = '/longxianda/sysProduct/list'
      getAction(httpurl, { pageNo: 1, pageSize: 10000 })
        .then(res => {
          if (res.success) {
            that.produceCodeList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取所有工序列表
    // loadProcedureList() {
    //   var that = this
    //   let httpurl = '/longxianda/sysProcedure/list'
    //   getAction(httpurl, { pageNo: 1, pageSize: 10000, order: 'desc' })
    //     .then(res => {
    //       if (res.success) {
    //         that.ProcedureList = res.result.records
    //       } else {
    //         that.$message.warning(res.message)
    //       }
    //     })
    //     .finally(() => {
    //       that.confirmLoading = false
    //     })
    // },
    // 根据产品id获取工序列表
    loadProcedureList(id) {
      var that = this
      let httpurl = '/longxianda/sysLine/queryById?id='
      // getAction(httpurl, { pageNo: 1, pageSize: 10000, order: 'desc' })
      getAction(httpurl + id)
        .then(res => {
          if (res.success) {
            that.ProcedureList = res.result.lineInfos
            // console.log(' that.ProcedureList', that.ProcedureList)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取计价方法
    loadvaluationList() {
      var that = this
      let httpurl = '/sys/api/queryEnableDictItemsByCode?code=valuation_method'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          //debugger
          if (res.length > 0) {
            that.valuationList = res
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
        }
      })
    }
  }
}
</script>
