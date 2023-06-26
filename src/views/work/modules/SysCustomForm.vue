<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="自定义类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="targeType">
              <!-- <a-input v-model="model.targetYpe" placeholder="请输入自定义类型（产品、工序、工单、报工、出入库单、出库表明细、用户）"  ></a-input> -->
              <a-select
                defaultValue="alipay"
                v-model="model.targetType"
                placeholder="请输入自定义类型（产品、工序、工单）"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option
                  v-for="(item, index) in recordList"
                  :key="index"
                  :value="item.value"
                  :label="item.label"
                  >{{ item.text }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="字段名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入字段名称"></a-input>
            </a-form-model-item>
          </a-col>

          <a-col :span="24">
            <a-form-model-item label="字段类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="type">
              <!-- <a-input v-model="model.type" placeholder="请输入字段类型"></a-input> -->
              <a-select
                defaultValue="alipay"
                v-model="model.type"
                @change="txtType"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option v-for="(item, index) in typeList" :key="index" :value="item.name">{{
                  item.name
                }}</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-model-item label="字段标识" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fieldName">
              <a-input v-model="model.fieldName" placeholder="请输入字段标识"></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24" v-if="model.type == '文本' || model.type == '大文本' || model.type == '数字'">
            <a-form-model-item label="引导文字" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="tips">
              <a-input v-model="model.tips" placeholder="请输入引导文字"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.type == '文本' || model.type == '大文本' || model.type == '数字'">
            <a-form-model-item label="提示说明" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="description">
              <a-input v-model="model.description" placeholder="请输入提示说明"></a-input>
            </a-form-model-item>
          </a-col>
          <!--          <a-col :span="24">
            <a-form-model-item label="排序" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sort">
              <a-input v-model="model.sort" placeholder="请输入排序"></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col
            :span="24"
            v-if="
              model.type == '文本' ||
                model.type == '大文本' ||
                model.type == '附件' ||
                model.type == '单选框' ||
                model.type == '复选框' ||
                model.type == '数字' ||
                model.type == '时间'
            "
          >
            <a-form-model-item label="字段属性" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="blRequired">
              <a-radio-group v-model="model.blRequired">
                <a-radio value="1">必填</a-radio>
                <a-radio
                  value="2"
                  v-if="model.type == '文本' || model.type == '大文本' || model.type == '数字' || model.type == '时间'"
                  >只读
                </a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-model-item label="必选" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="blRequired">
              <a-input v-model="model.blRequired" placeholder="请输入必选"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="只读" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="blReadonly">
              <a-input v-model="model.blReadonly" placeholder="请输入只读"></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24" v-if="model.type == '文本' || model.type == '大文本' || model.type == '数字'">
            <a-form-model-item label="默认值" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defaultValue">
              <a-input v-model="model.defaultValue" placeholder="请输入默认值"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="model.type == '单选框' || model.type == '复选框'">
            <a-form-model-item label="可选择值" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="options">
              <!--   <a-input v-model="model.options" placeholder="请输入可选择值"></a-input> -->
              <template>
                <div class="ant-table ant-table-empty" style="border: 1px solid #e8e8e8;">
                  <div class="ant-table-container">
                    <div class="ant-table-content">
                      <table style="table-layout: auto;">
                        <thead class="ant-table-thead">
                          <tr>
                            <th class="ant-table-cell">
                              操作
                            </th>

                            <th class="ant-table-cell">可选范围</th>
                          </tr>
                        </thead>
                        <tbody class="ant-table-tbody" v-if="listCont.length == 0">
                          <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                            <td colspan="4" class="ant-table-cell">
                              <div class="ant-empty ant-empty-normal">
                                <div class="ant-empty-image"></div>
                                <div class="ant-empty-description">暂无数据</div>
                              </div>
                            </td>
                          </tr>
                        </tbody>
                        <tbody class="ant-table-tbody" v-else>
                          <tr v-for="(item, index) in listCont" :key="index">
                            <td class="ant-table-cell" style="color: rgb(24,144,255);" @click="delNext(index)">
                              删除
                            </td>

                            <td class="ant-table-cell">
                              <a-input v-model="item.report" placeholder="请填写"></a-input>
                              <!-- <a-input v-else v-model="item.report" placeholder="请填写"></a-input> -->
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                  </div>
                  <div class="ant-table-footer">
                    <a class="link__21LS_ link-primary__1s7Ws" style="white-space: nowrap; cursor: pointer;">
                      <span
                        role="img"
                        type="plus-circle-o"
                        class="anticon anticon-plus-circle"
                        style="margin-right: 4px; padding: 0px; font-size: 14px;"
                        @click="addNext()"
                      >
                        添加一行
                      </span>
                    </a>
                  </div>
                </div>
              </template>
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
  name: 'SysCustomForm',
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
        xs: {
          span: 24
        },
        sm: {
          span: 3
        }
      },
      wrapperCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 20
        }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysCustom/add',
        edit: '/longxianda/sysCustom/edit',
        queryById: '/longxianda/sysCustom/queryById?id=',
        queryEnableDictItemsByCode: '/sys/api/queryEnableDictItemsByCode'
      },
      recordList: [],
      typeList: [
        {
          name: '文本'
        },
        {
          name: '大文本'
        },
        {
          name: '数字'
        },
        {
          name: '时间'
        },
        {
          name: '单选框'
        }
      ],
      listCont: []
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
      if (!this.model.id) {
        getAction(this.url.queryById + record.id).then(res => {
          this.model = Object.assign({}, res.result)
        })
      } else {
        this.model = Object.assign({}, record)
      }
      this.visible = true
    },
    submitForm() {
      const that = this
      //debugger
      var optionsList = []
      this.listCont.map(item => {
        optionsList.push(item.report)
      })
      if (this.listCont.length > 0) {
        this.model.options = JSON.stringify(optionsList)
      }
      this.model.fieldName = this.model.name
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
    queryEnableDictItemsByCode() {
      const that = this
      let smsParams = {
        code: 'custom'
      }
      getAction(this.url.queryEnableDictItemsByCode, smsParams, 'get')
        .then(res => {
          let record = []
          record.push(res[0])
          record.push(res[1])
          record.push(res[6])
          that.recordList = record
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    txtType() {
      let type = this.model.type
      if (type === '文本') {
      } else if (type === '大文本') {
      } else if (type === '数字') {
      } else if (type === '时间') {
      } else if (type === '附件') {
      } else if (type === '单选框') {
      } else if (type === '复选框') {
      }
    },
    addNext(e) {
      var listCont = this.listCont
      var pd = {
        type: this.model.type,
        report: null
      }
      listCont.push(pd)
    },
    delNext(index) {
      var listCont = this.listCont
      listCont.splice(index, 1)
    }
  }
}
</script>
