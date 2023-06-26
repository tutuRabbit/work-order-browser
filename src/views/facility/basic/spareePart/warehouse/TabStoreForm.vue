<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <!-- <a-col :span="24">
            <a-form-model-item label="删除标志" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="delFlag">
              <a-input-number v-model="model.delFlag" placeholder="请输入删除标志" style="width: 100%" />
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <a-form-model-item label="名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="storeName">
              <a-input v-model="model.storeName" placeholder="请输入名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="负责人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="storeManager">
              <a-input v-model="model.storeManager" placeholder="请输入负责人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="storePhone">
              <a-input v-model="model.storePhone" placeholder="请输入电话"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="storeAddress">
              <a-input v-model="model.storeAddress" placeholder="请输入地址"></a-input>
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-model-item label="入库类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="inStockType">
              <j-dict-select-tag
                type="list"
                dictCode="in_stock_type"
                v-model="model.inStockType"
                placeholder="请选择入库类型"
              ></j-dict-select-tag>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="出库类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outStockType">
              <j-dict-select-tag v-model="model.outStockType" dictCode="out_stock_type" placeholder="请选择出库类型">
              </j-dict-select-tag>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input v-model="model.remark" type="textarea" placeholder="请输入备注"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'

export default {
  name: 'TabStoreForm',
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
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {
        storeName: { required: true, message: '请输入名称!' },
        storePhone: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }]
      },
      url: {
        add: '/stock/tabStore/add',
        edit: '/stock/tabStore/edit',
        queryById: '/stock/tabStore/queryById'
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
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      getAction('/stock/tabStore/queryById?id=' + record.id).then(res => {
        if (res.success) {
          console.log('res.searchList', res.result)
          this.model = res.result
        } else {
          this.$message.warning(res.message)
        }
      })
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpUrl = ''
          let method = ''
          if (!this.model.id) {
            httpUrl += this.url.add
            method = 'post'
          } else {
            httpUrl += this.url.edit
            method = 'put'
          }
          console.log('httpUrl', httpUrl)
          httpAction(httpUrl, this.model, method)
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
