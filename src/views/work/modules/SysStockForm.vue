<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="产品编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code">
              <a-input v-model="model.code" placeholder="请输入产品编码"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName">
              <a-input v-model="model.productName" placeholder="请输入产品名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="库存单位"
              :rules="[{ required: true, message: '库存单位不能为空!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="stockPiece"
            >
              <a-select defaultValue="alipay" placeholder="请选择" v-model="model.stockPiece">
                <a-select-option v-for="(item, index) in stockPieceList" :value="item.id"
                  >{{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productNorms">
              <a-input v-model="model.productNorms" placeholder="请输入产品规格"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="产品属性"
              :rules="[{ required: true, message: '请输入产品属性!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="productProperty"
            >
              <a-select
                style="width: 100%"
                v-model="model.productProperty"
                placeholder="请输入产品属性"
                :allowClear="true"
              >
                <a-select-option v-for="(action, index) in textList" :key="index" :value="action.value">{{
                  action.label
                }}</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工艺路线" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="craftRoute">
              <a-select defaultValue="alipay" placeholder="请选择" v-model="model.craftRoute">
                <a-select-option v-for="(item, index) in LineList" :value="item.name">{{ item.name }} </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="最大库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="maxStock">
              <a-input-number v-model="model.maxStock" placeholder="请输入最大库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="最小库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="minStock">
              <a-input-number v-model="model.minStock" placeholder="请输入最小库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="安全库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="securityStock">
              <a-input-number v-model="model.securityStock" placeholder="请输入安全库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="库存数量" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockNumber">
              0
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
  name: 'SysProductForm',
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
        code: [{ required: true, message: '请输入产品编码!' }]
      },
      url: {
        add: '/longxianda/sysProduct/add',
        edit: '/longxianda/sysProduct/edit',
        queryById: '/longxianda/sysProduct/queryById'
      },
      stockPieceList: null,
      LineList: null,
      textList: [
        { label: '自制', value: '自制', defaultChecked: true },
        { label: '外购', value: '外购', defaultChecked: false }
      ]
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
    this.loadStockPieceList()
    this.loadLineList()
  },
  methods: {
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      this.visible = true
    },
    // 获取单位列表
    loadStockPieceList() {
      var that = this
      let httpurl = '/longxianda/sysSpec/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.stockPieceList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取工艺路线列表
    loadLineList() {
      var that = this
      let httpurl = '/longxianda/sysLine/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          //debugger
          if (res.success) {
            that.LineList = res.result.records
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
