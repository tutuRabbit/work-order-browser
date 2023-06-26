<template>
  <j-modal
    title="添加外委"
    :width="width"
    :fullscreen="fullscreen"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <!-- <a-form > -->
      <a-form-model ref="form" :model="model" :rules="validatorRules">
        <a-row :gutter="24">
          <a-col :span="24"><div class="title" style="margin:6px 0">设备信息</div></a-col>
          <a-col :span="8">
            <a-form-model-item label="维修单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input :disabled="true" v-model.trim="model.repairOrderNo" placeholder="不填可自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input :disabled="true" v-model.trim="model.equipmentCode" placeholder="请输入设备编码"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input :disabled="true" v-model.trim="model.equipmentName" placeholder="请输入设备名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input :disabled="true" v-model.trim="model.equipmentType"></a-input>
              <!-- <j-category-select
                :disabled="true"
                v-model.trim="model.equipmentType"
                pcode="B03"
                placeholder="请选择设备类型"
              /> -->
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input :disabled="true" v-model.trim="model.equipmentModel" placeholder="请输入规格型号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24"><div class="title" style="margin:10px 0">外委维修</div></a-col>
          <a-col :span="8">
            <a-form-model-item label="外委单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-model.trim="model.workOrderId" placeholder="不填可自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="外委单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <!-- <a-input v-model.trim="model.entrustCompany" placeholder="请输入外委单位"></a-input> -->
              <a-select
                :showSearch="true"
                v-model="model.entrustCompany"
                placeholder="回车可搜索"
                style="width: 100%"
                optionFilterProp="label"
              >
                <a-select-option
                  v-for="(item, index) in companyOptions"
                  :key="index"
                  :value="item.id"
                  :label="item.companyName"
                >
                  {{ item.companyName }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-model.trim="model.entrustContact" placeholder="请输入联系人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="联系方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="entrustTel">
              <a-input v-model.trim="model.entrustTel" placeholder="请输入联系方式"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="预估费用" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-model.trim="model.predictCost" placeholder="请输入预估费用"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="申请原因" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="applyReason">
              <a-input type="textarea" v-model.trim="model.applyReason" placeholder="请输入申请原因"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
      <!-- </a-form> -->
    </div>
  </j-modal>
</template>

<script>
import TroubleFixReportForm from './TroubleFixReportForm'
import { postAction, getAction } from '@api/manage'

export default {
  name: 'TroubleFixReportModal',
  components: {
    TroubleFixReportForm
  },
  data() {
    return {
      companyOptions: [],
      title: '',
      width: '80%',
      visible: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      model: {},
      queryParam: {},
      validatorRules: {
        entrustTel: [{ required: false, message: '请输入手机号!' }, { validator: this.validatePhone }],
        applyReason: [{ required: true, message: '请输入申请原因!' }, { validator: this.validatePhone }]
      },
      fullscreen: true
    }
  },
  created() {
    // 往来单位
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      // getAction('/equipment/tabContactCompany/list').then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        console.log('res.供应商', res.result)
        this.companyOptions = arr
      }
    })
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
    editor(row) {
      console.log('添加外委', row)
      this.model = Object.assign({}, row)
      // this.queryParam.id = row.id
      this.visible = true
    },
    close() {
      this.visible = false
      this.queryParam = {}
    },
    handleOk() {
      console.log('this.model', this.model)
      this.$refs.form.validate(valid => {
        if (valid) {
          // if (this.model.applyReason) {
          delete this.model.fixWorkOrderWorkloads
          // 新增外委的接口
          postAction('/fixworkorder/outFixEntrust/add', this.model).then(res => {
            if (res.success) {
              if (res.code == 200) {
                this.visible = false
                this.$message.success(res.message)
                that.$emit('ok')
              }
            } else {
              this.$message.warning(res.message)
            }
            this.visible = false
          })
          // } else {
          //   this.$message.warning('请输入申请原因！')
          // }
        }
      })
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
