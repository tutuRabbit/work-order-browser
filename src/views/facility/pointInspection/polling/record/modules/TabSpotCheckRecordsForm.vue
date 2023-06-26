<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled" class="table-page-search-wrapper">
      <a-form-model layout="inline" ref="form" :model="model" :rules="validatorRules" slot="detail">
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
            <a-form-model-item label="点检时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spotCheckTime">
              <j-date
                placeholder="请选择点检时间"
                v-model="model.spotCheckTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="7" style="margin-left:10px">
            <a-form-model-item label="点检人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spotCheckPerson">
              <j-dict-select-tag
                v-model="model.spotCheckPerson"
                placeholder="请选择点检人"
                dictCode="sys_user,realname,id"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="patrolItem">
              <a-input v-model="model.remark" placeholder="请输入备注" id="describe" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
      <j-editable-table
        :columns="SpotTab.columns"
        :dataSource="SpotTab.dataSource"
        :rowNumber="true"
        :rowSelection="false"
        :maxHeight="400"
      />
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { FormTypes } from '@/utils/JEditableTableUtil'
import JEditableTable from '@/components/jeecg/JEditableTable'

export default {
  name: 'TabSpotCheckRecordsForm',
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
      SpotVis: false,
      SpotData: {},
      SpotTab: {
        dataSource: [
          { xunjianxiangmu: 'hello', select: 'int', checkbox: true, datetime: '2019-6-17 14:50:48' },
          { xunjianxiangmu: 'world', select: 'string', checkbox: false, datetime: '2019-6-16 14:50:48' },
          { xunjianxiangmu: 'one', select: 'double', checkbox: true, datetime: '2019-6-17 15:50:48' },
          { xunjianxiangmu: 'two', select: 'boolean', checkbox: false, datetime: '2019-6-14 14:50:48' },
          { xunjianxiangmu: 'three', select: '', checkbox: false, datetime: '2019-6-13 14:50:48' }
        ],
        columns: [
          {
            title: '标准名称',
            align: 'center',
            type: FormTypes.normal,
            key: 'xunjianxiangmu'
          },
          {
            title: '上限',
            align: 'center',
            type: FormTypes.normal,
            key: 'status'
          },
          {
            title: '下限',
            align: 'center',
            type: FormTypes.normal,
            key: 'loujiannum'
          },
          {
            title: '巡检记录',
            align: 'center',
            type: FormTypes.normal,
            key: 'yichangnum'
          },
          {
            title: '巡检结果',
            align: 'center',
            type: FormTypes.normal,
            key: 'equmentCode'
          },
          {
            title: '是否异常',
            align: 'center',
            type: FormTypes.normal,
            key: 'equmentName'
          },
          {
            title: '最后巡检人',
            align: 'center',
            type: FormTypes.normal,
            key: 'type'
          },
          {
            title: '巡检人',
            align: 'center',
            type: FormTypes.normal,
            key: 'person'
          },
          {
            title: '备注',
            align: 'center',
            type: FormTypes.input,
            key: 'remark',
            placeholder: '清输入'
          }
        ]
      },
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
      validatorRules: {},
      url: {
        add: '/spot_record/tabSpotCheckRecords/add',
        edit: '/spot_record/tabSpotCheckRecords/edit',
        queryById: '/spot_record/tabSpotCheckRecords/queryById'
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
      console.log('record', record)
      // this.model = Object.assign({}, record)
      getAction(this.url.queryById, { id: record.id }).then(res => {
        this.model = Object.assign({}, res.result)
        console.log(' res.result', res.result)
      })
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
