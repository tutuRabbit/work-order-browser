<template>
  <a-drawer
      :title="title"
      :width="800"
      placement="right"
      :closable="false"
      @close="close"
      :visible="visible">

    <a-spin :spinning="confirmLoading">
      <a-form-model ref="form" :model="model" :rules="validatorRules">
      
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="userId" label="生产人ID">
          <a-input placeholder="请输入生产人ID" v-model="model.userId" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="userName" label="生产人名称">
          <a-input placeholder="请输入生产人名称" v-model="model.userName" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processCode" label="工序编号">
          <a-input placeholder="请输入工序编号" v-model="model.processCode" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processName" label="工序名称">
          <a-input placeholder="请输入工序名称" v-model="model.processName" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productCode" label="产品编号">
          <a-input placeholder="请输入产品编号" v-model="model.productCode" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName" label="产品名称">
          <a-input placeholder="请输入产品名称" v-model="model.productName" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productSpec" label="产品规格">
          <a-input placeholder="请输入产品规格" v-model="model.productSpec" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productUnit" label="产品单位">
          <a-input placeholder="请输入产品单位" v-model="model.productUnit" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fineAmount" label="良品数">
          <a-input placeholder="请输入良品数" v-model="model.fineAmount" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectsAmount" label="不良品数">
          <a-input placeholder="请输入不良品数" v-model="model.defectsAmount" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="defectRatio" label="defectRatio">
          <a-input-number v-model="model.defectRatio" />
        </a-form-model-item>
        <a-form-model-item :labelCol="labelCol" :wrapperCol="wrapperCol" prop="totalAmount" label="报工数">
          <a-input placeholder="请输入报工数" v-model="model.totalAmount" />
        </a-form-model-item>
		
      </a-form-model>
    </a-spin>

    <div class="drawer-bootom-button">
      <a-button type="primary" @click="handleOk">确定</a-button>
      <a-button type="primary" @click="handleCancel">取消</a-button>
    </div>
  </a-drawer>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import moment from "moment"

  export default {
    name: "PperformanceModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        validatorRules:{
        },
        url: {
          add: "/longxianda2/pperformance/add",
          edit: "/longxianda2/pperformance/edit",
        },
      }
    },
    created () {
    },
    methods: {
      add () {
        //初始化默认值
        this.edit({});
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.$refs.form.clearValidate();
      },
      handleOk () {
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
              that.close();
            })
          }else{
            return false;
         }
        })
      },
      handleCancel () {
        this.close()
      },


    }
  }
</script>

<style lang="less" scoped>
  /**Button按钮间距*/
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
 /**抽屉按钮样式*/
  .drawer-bootom-button {
    position: absolute;
    bottom: -8px;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }
</style>