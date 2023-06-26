<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="8">
            <a-form-model-item label="维修单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixOrderNo">
              <a-input v-model="model.fixOrderNo" placeholder="请输入维修单号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentNo">
              <a-input v-model="model.equipmentNo" placeholder="请输入设备编码"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentName">
              <a-input v-model="model.equipmentName" placeholder="请输入设备名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentModel">
              <a-input v-model="model.equipmentModel" placeholder="请输入规格型号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="使用部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="useDepart">
              <a-input v-model="model.useDepart" placeholder="请输入使用部门"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentType">
              <a-input v-model="model.equipmentType" placeholder="请输入设备类型"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="申请人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="applicant">
              <a-input v-model="model.applicant" placeholder="请输入申请人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="startTime">
              <j-date
                placeholder="请选择维修开始时间"
                v-model="model.startTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endTime">
              <j-date
                placeholder="请选择维修结束时间"
                v-model="model.endTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修用时（小时）" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spendTime">
              <a-input v-model="model.spendTime" placeholder="请输入维修用时（小时）"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="停机时长（小时）" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stopTime">
              <a-input v-model="model.stopTime" placeholder="请输入停机时长（小时）"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="备件费用" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareCost">
              <a-input v-model="model.spareCost" placeholder="请输入备件费用"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修级别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixLevel">
              <a-input v-model="model.fixLevel" placeholder="请输入维修级别"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="故障原因" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="errorReason">
              <!-- <a-input v-model="model.errorReason" placeholder="请输入故障原因"></a-input> -->
              <j-dict-select-tag
                type="list"
                v-model="model.errorReason"
                dictCode="failure_cause"
                placeholder="请选择故障原因"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixGroup">
              <!-- <a-input v-model="model.fixGroup" placeholder="请输入维修组"></a-input> -->
              <j-dict-select-tag
                type="list"
                v-model="model.fixGroup"
                dictCode="maintain_list"
                placeholder="请选择维修组"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修人员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixWorker">
              <a-input v-model="model.fixWorker" placeholder="请输入维修人员"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="验证人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="verifiedBy">
              <a-input v-model="model.verifiedBy" placeholder="请输入验证人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="评分" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="score">
              <a-input v-model="model.score" placeholder="请输入评分"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修评价" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="evaluate">
              <a-input v-model="model.evaluate" placeholder="请输入维修评价"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="外委单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outEntrustCompany">
              <a-input v-model="model.outEntrustCompany" placeholder="请输入外委单位"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修费用" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixCost">
              <a-input v-model="model.fixCost" placeholder="请输入维修费用"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="故障描述" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="errorDescription">
              <a-input v-model="model.errorDescription" placeholder="请输入故障描述"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="工作描述" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workDescription">
              <a-input v-model="model.workDescription" placeholder="请输入工作描述"></a-input>
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
  name: 'FixHistoryForm',
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
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/fixHistory/fixHistory/add',
        edit: '/fixHistory/fixHistory/edit',
        queryById: '/fixHistory/fixHistory/queryById'
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
