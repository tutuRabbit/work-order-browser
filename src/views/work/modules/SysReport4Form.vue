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
            <a-form-model-item label="产品单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productUnit">
              <a-input v-model="model.productUnit" placeholder="请输入产品单位"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productSpec">
              <a-input v-model="model.productSpec" placeholder="请输入产品规格"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单计划数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workPlanAmount">
              <a-input v-model="model.workPlanAmount" placeholder="请输入工单计划数"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单实际数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workRealaMount">
              <a-input v-model="model.workRealaMount" placeholder="请输入工单实际数"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workStatus">
              <a-input v-model="model.workStatus" placeholder="请输入工单状态"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单计划开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workStartTime">
              <a-input v-model="model.workStartTime" placeholder="请输入工单计划开始时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单计划结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workEndTime">
              <a-input v-model="model.workEndTime" placeholder="请输入工单计划结束时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单实际开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workRealStartTime">
              <a-input v-model="model.workRealStartTime" placeholder="请输入工单实际开始时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单实际结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workRealEndTime">
              <a-input v-model="model.workRealEndTime" placeholder="请输入工单实际结束时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工单备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="workRemark">
              <a-input v-model="model.workRemark" placeholder="请输入工单备注"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序顺序" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processOrder">
              <a-input v-model="model.processOrder" placeholder="请输入工序顺序"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processName">
              <a-input v-model="model.processName" placeholder="请输入工序名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processCode">
              <a-input v-model="model.processCode" placeholder="请输入工序编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工数配比" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputRate">
              <a-input v-model="model.outputRate" placeholder="请输入报工数配比"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskStatus">
              <a-input v-model="model.taskStatus" placeholder="请输入任务状态"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务计划开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskStartTime">
              <a-input v-model="model.taskStartTime" placeholder="请输入任务计划开始时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务计划结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskEndTime">
              <a-input v-model="model.taskEndTime" placeholder="请输入任务计划结束时间"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务实际开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskRealStartTime">
              <j-date placeholder="请选择任务实际开始时间" v-model="model.taskRealStartTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务实际结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskRealEndTime">
              <j-date placeholder="请选择任务实际结束时间" v-model="model.taskRealEndTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务计划数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskPlanAmount">
              <a-input-number v-model="model.taskPlanAmount" placeholder="请输入任务计划数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="任务实际数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="taskRealAmount">
              <a-input-number v-model="model.taskRealAmount" placeholder="请输入任务实际数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="生产人员" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operator">
              <a-input v-model="model.operator" placeholder="请输入生产人员"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="标准效率" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="standardEfficiency">
              <a-input v-model="model.standardEfficiency" placeholder="请输入标准效率"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="实际效率" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="actualEfficiency">
              <a-input v-model="model.actualEfficiency" placeholder="请输入实际效率"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="达标率" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="controlRate">
              <a-input-number v-model="model.controlRate" placeholder="请输入达标率" style="width: 100%" />
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
            <a-form-model-item label="不良品数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectsAmount">
              <a-input-number v-model="model.defectsAmount" placeholder="请输入不良品数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="不良品原因" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectsstr">
              <a-input v-model="model.defectsstr" placeholder="请输入不良品原因"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reporStartTime">
              <j-date placeholder="请选择报工开始时间" v-model="model.reporStartTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reporEndTime">
              <j-date placeholder="请选择报工结束时间" v-model="model.reporEndTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="实际报工时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reporRealTime">
              <j-date placeholder="请选择实际报工时间" v-model="model.reporRealTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="报工时长" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputDuration">
              <a-input-number v-model="model.outputDuration" placeholder="请输入报工时长" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="计价方式" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="pricingType">
              <a-input v-model="model.pricingType" placeholder="请输入计价方式"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工资单价" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="unitPrice">
              <a-input-number v-model="model.unitPrice" placeholder="请输入工资单价" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="预计工资	" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="expectedSalary">
              <a-input-number v-model="model.expectedSalary" placeholder="请输入预计工资	" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审批人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkor">
              <a-input v-model="model.checkor" placeholder="请输入审批人"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审批时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkTime">
              <j-date placeholder="请选择审批时间" v-model="model.checkTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审批状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkStatus">
              <a-input v-model="model.checkStatus" placeholder="请输入审批状态"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input v-model="model.remark" placeholder="请输入备注"  ></a-input>
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
    name: 'SysReport4Form',
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
          add: "/longxianda2/sysReport4/add",
          edit: "/longxianda2/sysReport4/edit",
          queryById: "/longxianda2/sysReport4/queryById"
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