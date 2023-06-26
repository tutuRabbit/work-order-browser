<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="工单编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="projectCode">
              <a-input v-model="model.projectCode" placeholder="请输入工单编码"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productCode">
              <a-input v-model="model.productCode" placeholder="请输入产品编码"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName">
              <a-input v-model="model.productName" placeholder="请输入产品名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productSpec">
              <a-input v-model="model.productSpec" placeholder="请输入产品规格"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processCode">
              <a-input v-model="model.processCode" placeholder="请输入工序编码"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processName">
              <a-input v-model="model.processName" placeholder="请输入工序名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="生产人员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operatorName">
              <a-input v-model="model.operatorName" placeholder="请输入生产人员"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planStartTime">
              <j-date placeholder="请选择报工开始时间" v-model="model.planStartTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planEndTime">
              <j-date placeholder="请选择报工结束时间" v-model="model.planEndTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="不良品项" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectName">
              <a-input v-model="model.defectName" placeholder="请输入不良品项"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="计划数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planAmount">
              <a-input-number v-model="model.planAmount" placeholder="请输入计划数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputAmount">
              <a-input-number v-model="model.outputAmount" placeholder="请输入报工数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="良品数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fineAmount">
              <a-input-number v-model="model.fineAmount" placeholder="请输入良品数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="不良品数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectAmount">
              <a-input-number v-model="model.defectAmount" placeholder="请输入不良品数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="不良品率" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectRatio">
              <a-input-number v-model="model.defectRatio" placeholder="请输入不良品率" style="width: 100%" />
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
    name: 'SysReport1Form',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        model:{
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/longxianda/sysReport1/add",
          edit: "/longxianda/sysReport1/edit",
          queryById: "/longxianda/sysReport1/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
    }
  }
</script>