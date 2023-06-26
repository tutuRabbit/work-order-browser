<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :fullscreen="fullscreen"
    :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
    @cancel="close"
    cancelText="关闭"
  >
    <div class="table-page-search-wrapper">
      <a-form-model ref="form" :model="model" :rules="validatorRules">
        <a-row>
          <a-col :span="7" style="margin-left:10px">
            <a-form-model-item label="设备编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="deviceCode">
              <a-input disabled v-model="model.deviceCode" placeholder="请输入设备编号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="7" style="margin-left:10px">
            <a-form-model-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="deviceName">
              <a-input disabled v-model="model.deviceName" placeholder="请输入设备名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="7" style="margin-left:10px">
            <a-form-model-item
              label="点检时间"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="spotCheckTime"
              :rules="[{ required: true, message: '请选择点检时间!' }]"
            >
              <j-date
                placeholder="请选择点检时间"
                v-model="model.spotCheckTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
            <!-- <a-form-model-item label="点检时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spotCheckTime">
              <j-date
                placeholder="请选择点检时间"
                v-model="model.spotCheckTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item> -->
          </a-col>
          <a-col :span="7" style="margin-left:10px">
            <a-form-model-item
              label="点检人"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="spotCheckPerson"
              :rules="[{ required: true, message: '请选择点检人!' }]"
            >
              <j-dict-select-tag
                v-model="model.spotCheckPerson"
                placeholder="请选择点检人"
                dictCode="sys_user,realname,id"
              />
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="7" style="margin-left:10px">
            <a-form-model-item label="巡检项目" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="patrolItem">
              <a-input v-model="model.patrolItem" placeholder="请输入巡检项目"></a-input>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="{ span: 2 }" :wrapperCol="{ span: 20 }">
              <a-textarea :rows="5" v-model="model.remark" placeholder="请输入备注" id="remark" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </div>
    <template slot="footer">
      <a-button @click="close">关闭</a-button>
      <a-button type="primary" @click="submitForm">确认</a-button>
    </template>
  </j-modal>
</template>

<script>
import { postAction } from '@/api/manage'
import { FormTypes } from '@/utils/JEditableTableUtil'
export default {
  name: 'recordModal',
  components: {
    // TabSpotCheckItemsForm
  },
  data() {
    return {
      title: '添加记录',
      width: 800,
      fullscreen: true,
      visible: false,
      confirmLoading: false,
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '标准名称',
            key: 'deviceCode',
            width: '18%',
            type: FormTypes.normal
          },
          {
            title: '上限',
            key: 'deviceName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '下限',
            key: 'ss',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '参考值',
            key: 'spotCheckPerson',
            width: '18%',
            type: FormTypes.select,
            options: [
              { title: '男', value: '1' },
              { title: '女', value: '2' }
            ],
            defaultValue: '',
            placeholder: '请选择${title}'
          },
          {
            title: '巡检结果',
            key: 'patrolItem',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '是否异常',
            key: 'A',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '备注',
            // key: 'd',
            maxWidth: 500,
            ellipsis: true,
            width: '10%',
            type: FormTypes.normal
          }
        ]
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 8 }
      },
      model: {},
      validatorRules: {
        // spotCheckTime: {
        // }
      },
      disableSubmit: false
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
      let res = record[0]
      this.model = {
        spotCheckItemId: res.id,
        deviceCode: res.deviceCode,
        deviceName: res.deviceName
      }
      this.visible = true
    },
    close() {
      this.visible = false
      this.model = {}
    },
    // /spot_record/tabSpotCheckRecords/add
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        console.log('valid', valid)
        if (valid) {
          that.confirmLoading = true
          postAction('/spot_record/tabSpotCheckRecords/add', that.model)
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
              that.visible = false
            })
        }
      })
      // if (this.model.spotCheckTime == '' || this.model.spotCheckTime == undefined) {
      //   that.$message.warning('请选择点检时间！')
      // } else if (this.model.spotCheckPerson == '' || this.model.spotCheckPerson == undefined) {
      //   that.$message.warning('请输入点检人!')
      // } else {
      //   that.confirmLoading = true
      //   postAction('/spot_record/tabSpotCheckRecords/add', that.model)
      //     .then(res => {
      //       if (res.success) {
      //         that.$message.success(res.message)
      //         that.$emit('ok')
      //       } else {
      //         that.$message.warning(res.message)
      //       }
      //     })
      //     .finally(() => {
      //       that.confirmLoading = false
      //       that.visible = false
      //     })
      // }
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>
