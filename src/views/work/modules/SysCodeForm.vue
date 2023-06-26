<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="目标表单名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="type">
              <a-select defaultValue="alipay" v-model="model.type" :showSearch="true" optionFilterProp="label">
                <a-select-option
                  v-for="(item, index) in recordList"
                  :value="item.value"
                  :key="index"
                  :label="item.label"
                  >{{ item.text }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="编号前缀" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="prefix">
              <a-input v-model="model.prefix" placeholder="请输入编号前缀" @input="txtprefix"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="提交时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="submitTime">
              <a-radio-group @change="txtprefix" v-model="model.submitTime">
                <a-radio value="年">年</a-radio>
                <a-radio value="年月">年月</a-radio>
                <a-radio value="年月日">年月日</a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="流水号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="serial">
              <a-input-number
                v-model="model.serial"
                style="width: 150px;"
                @change="txtprefix"
                :min="1"
                :max="8"
                placeholder="请输入流水号"
              ></a-input-number
              >&nbsp;位 &nbsp;例：{{ this.sizeList }}
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="表单编码生成规则" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="rule">
              <a-input v-model="model.rule" placeholder="请输入表单编码生成规则"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <div
              style="border: 1px solid #d5d5d5;
                    height: 50px;
                    line-height: 50px;
                    font-size: 18px;
                    margin: 0 20px;
                    padding-left: 40px;
                    font-size:18px;
                    "
            >
              <span>编号预览：{{ numbers }}</span>
            </div>
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
  name: 'SysCodeForm',
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
      sizeList: '9',
      labelCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 5
        }
      },
      wrapperCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 16
        }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysCode/add',
        edit: '/longxianda/sysCode/edit',
        queryById: '/longxianda/sysCode/queryById',
        queryEnableDictItemsByCode: '/sys/api/queryEnableDictItemsByCode'
      },
      recordList: [],
      numbers: ''
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
    this.queryEnableDictItemsByCode()
  },
  methods: {
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
      this.txtprefix()
    },
    queryEnableDictItemsByCode() {
      const that = this
      let smsParams = {
        code: 'code_type'
      }
      getAction(this.url.queryEnableDictItemsByCode, smsParams, 'get')
        .then(res => {
          that.recordList = res
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
    },
    txtprefix(e) {
      var serial = this.model.serial * 1
      if (serial > 8) {
        this.model.serial = ''
        return false
      }
      var number0 = ''
      var number9 = ''
      for (var i = 0; i < serial; i++) {
        number0 += '0'
        number9 += '8'
      }
      this.sizeList = number0
      this.model.rule = this.model.prefix + '+' + this.model.submitTime + '+' + number0 + '-' + number9
      var date = new Date() //实例一个时间对象；
      var time = ''
      if (this.model.submitTime == '年') {
        time = date.getFullYear()
      } else if (this.model.submitTime == '年月') {
        time = date.getFullYear() + '' + (date.getMonth() + 1)
      } else {
        time = date.getFullYear() + '' + (date.getMonth() + 1) + '' + date.getDate()
      }

      this.numbers = this.model.prefix + time + number0
    }
  }
}
</script>
