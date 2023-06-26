<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="单位编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="companyCode">
              <a-input v-model="model.companyCode" placeholder="请输入单位编码"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="单位名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="companyName">
              <a-input v-model="model.companyName" placeholder="请输入单位名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="默认联系人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defaultContact">
              <a-input v-model="model.defaultContact" placeholder="请输入默认联系人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="qq" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="qq">
              <a-input v-model.number="model.qq" placeholder="请输入qq"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="微信" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="wechat">
              <a-input v-model="model.wechat" placeholder="请输入微信"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="tel">
              <a-input v-model="model.tel" placeholder="请输入电话"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="邮箱" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="email">
              <a-input v-model="model.email" placeholder="请输入邮箱"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="传真" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fax">
              <a-input v-model="model.fax" placeholder="请输入传真"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="税务登记号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taxRegistration">
              <a-input v-model="model.taxRegistration" placeholder="请输入税务登记号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="address">
              <a-input v-model="model.address" placeholder="请输入地址"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="邮编" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="zipCode">
              <a-input v-model.number="model.zipCode" placeholder="请输入邮编"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="开户行" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="openingBank">
              <a-input v-model="model.openingBank" placeholder="请输入开户行"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="银行帐户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="bankAccount">
              <a-input v-model.number="model.bankAccount" placeholder="请输入银行帐户"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="网址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="website">
              <a-input v-model="model.website" placeholder="请输入网址"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="note">
              <a-input style="width:100%" type="textarea" v-model="model.note" placeholder="请输入备注"></a-input>
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
  name: 'TabContactCompanyForm',
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
        companyName: [
          { required: true, message: '请输入单位名称', min: 0, max: 50, message: '单位名称限制为50个字符以内' }
        ],
        tel: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }],
        companyCode: [{ min: 0, max: 20, message: '单位编码限制为20个字符以内' }],

        defaultContact: [{ min: 0, max: 50, message: '默认联系人限制为50个字符以内' }],
        // qq: [{ min: 0, max: 20, message: 'qq限制为20个字符以内' }],
        wechat: [{ min: 0, max: 20, message: '默认联系人限制为20个字符以内' }],
        email: [{ min: 0, max: 20, message: '邮箱限制为20个字符以内' }],
        fax: [{ min: 0, max: 10, message: '传真限制为10个字符以内' }],
        taxRegistration: [{ min: 0, max: 20, message: '税务登记号限制为20个字符以内' }],
        address: [{ min: 0, max: 50, message: '地址限制为50个字符以内' }],
        // zipCode: [{ min: 0, max: 6, message: '邮编限制为6个字符以内' }],
        openingBank: [{ min: 0, max: 30, message: '开户行限制为30个字符以内' }],
        // bankAccount: [{ min: 0, max: 20, message: '银行帐户限制为20个字符以内' }],
        website: [{ min: 0, max: 30, message: '网址限制为30个字符以内' }],
        note: [{ min: 0, max: 200, message: '备注限制为200个字符以内' }]
      },
      url: {
        add: '/equipment/tabContactCompany/add',
        edit: '/equipment/tabContactCompany/edit',
        queryById: '/equipment/tabContactCompany/queryById'
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
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
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
