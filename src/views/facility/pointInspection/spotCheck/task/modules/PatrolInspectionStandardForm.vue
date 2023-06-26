<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <!-- <a-col :span="24">
            <a-form-model-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">
              <a-select v-model="model.status" class="code-mode-select" placeholder="请选择状态">
                <a-select-option value="1">执行中</a-select-option>
                <a-select-option value="4">待执行</a-select-option>
                <a-select-option value="5">未开始</a-select-option>
                <a-select-option value="6">已完成</a-select-option>
          待验证5已完成6 
              </a-select>
            </a-form-model-item>
          </a-col> -->
          <a-col :span="12">
            <a-form-model-item label="计划名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planName">
              <a-input v-model="model.planName" placeholder="请输入计划名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="巡检时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planStartTime">
              <j-date
                placeholder="请选择巡检时间"
                v-model="model.planStartTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-model-item
              label="计划巡检结束时间"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="planEndTime"
            >
              <j-date
                placeholder="请选择计划巡检结束时间"
                v-model="model.planEndTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col> -->
          <a-col :span="12">
            <a-form-model-item label="巡检人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planExecutor">
              <a-input v-model="model.planExecutor" placeholder="请输入巡检人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="备注"
              :labelCol="{ xs: { span: 24 }, sm: { span: 3 } }"
              :wrapperCol="{ xs: { span: 24 }, sm: { span: 20 } }"
              prop="remark"
            >
              <a-input v-model="model.remark" placeholder="请输入备注"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-table
              size="middle"
              :scroll="{ x: true }"
              :bordered="false"
              rowKey="id"
              :columns="pollingTable.columns"
              :dataSource="pollingTable.dataSource"
              :loading="pollingTable.loading"
              :pagination="false"
              class="j-table-force-nowrap"
            >
              <span slot="action" slot-scope="text, record">
                <a @click="handleDetail(record)">详细</a>
              </span>
            </a-table>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
    <j-modal title="点巡检" width="55%" :visible="SpotVis" @ok="SpotOk" @cancel="SpotVis = false" cancelText="关闭">
      <!-- <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="SpotSearch">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="开始时间">
                <j-date
                  placeholder="请选择开始时间"
                  v-model="SpotData.planStartTime"
                  :show-time="true"
                  date-format="YYYY-MM-DD HH:mm:ss"
                  style="width: 100%"
                />
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="结束时间">
                <j-date
                  placeholder="请选择结束时间"
                  v-model="SpotData.planEndTime"
                  :show-time="true"
                  date-format="YYYY-MM-DD HH:mm:ss"
                  style="width: 100%"
                />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div> -->
      <j-editable-table
        ref="spotRef"
        :columns="SpotTab.columns"
        :dataSource="SpotTab.dataSource"
        :rowNumber="true"
        :rowSelection="false"
        :maxHeight="400"
      />
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { FormTypes } from '@/utils/JEditableTableUtil'
import JEditableTable from '@/components/jeecg/JEditableTable'

export default {
  name: 'PatrolInspectionStandardForm',
  components: { JEditableTable },
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
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      detailVis: false,
      pollingTable: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '操作',
            dataIndex: 'action',
            align: 'center',
            fixed: 'left',
            scopedSlots: { customRender: 'action' }
          },
          {
            title: '巡检项目',
            align: 'center',
            dataIndex: 'patrolProjectName'
          },
          {
            title: '合计项',
            align: 'center',
            dataIndex: 'aggregateTerm'
          },
          {
            title: '漏检数',
            align: 'center',
            dataIndex: 'missingNumber'
          },
          {
            title: '异常数',
            align: 'center',
            dataIndex: 'anomalyNumber'
          },
          {
            title: '设备编码',
            align: 'center',
            dataIndex: 'equipmentCode'
          },
          {
            title: '设备名称',
            align: 'center',
            dataIndex: 'equipmentName'
          },
          {
            title: '规格型号',
            align: 'center',
            dataIndex: 'equipmentModel'
          },
          {
            title: '执行人',
            align: 'center',
            dataIndex: 'person'
          }
        ],
        pagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        }
      },
      SpotVis: false,
      SpotData: {},
      SpotTab: {
        dataSource: [
          // { xunjianxiangmu: 'hello', select: 'int', checkbox: true, datetime: '2019-6-17 14:50:48' },
          // { xunjianxiangmu: 'world', select: 'string', checkbox: false, datetime: '2019-6-16 14:50:48' },
          // { xunjianxiangmu: 'one', select: 'double', checkbox: true, datetime: '2019-6-17 15:50:48' },
          // { xunjianxiangmu: 'two', select: 'boolean', checkbox: false, datetime: '2019-6-14 14:50:48' },
          // { xunjianxiangmu: 'three', select: '', checkbox: false, datetime: '2019-6-13 14:50:48' }
        ],
        columns: [
          {
            title: '标准名称',
            align: 'center',
            type: FormTypes.normal,
            key: 'name'
          },
          {
            title: '上限',
            align: 'center',
            type: FormTypes.normal,
            key: 'upperLimit'
          },
          {
            title: '下限',
            align: 'center',
            type: FormTypes.normal,
            key: 'lowerLimit'
          },
          {
            title: '巡检记录',
            align: 'center',
            // type: FormTypes.normal,
            type: FormTypes.input,
            key: 'inspectionRecord'
          },
          {
            title: '巡检结果',
            align: 'center',
            type: FormTypes.normal,
            key: 'inspectionResult'
          },
          {
            title: '是否异常',
            align: 'center',
            type: FormTypes.normal,
            key: 'normal'
          },
          {
            title: '最后巡检人',
            align: 'center',
            type: FormTypes.normal,
            key: 'lastInspector'
          },
          {
            title: '巡检人',
            align: 'center',
            type: FormTypes.normal,
            key: 'inspector'
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
      validatorRules: {},
      url: {
        add: '/checkplan/patrolInspectionStandard/add',
        edit: '/checkplan/patrolInspectionStandard/edit',
        queryById: '/checkplan/patrolInspectionStandard/queryById?id=',
        queryByEquipmentId: '/checkplan/patrolInspectionStandard/queryByEquipmentId?id=',
        editTask: '/checkplan/patrolInspectionStandard/editTask'
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
    SpotSearch() {},
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      if (record.id) {
        getAction(this.url.queryById + record.id).then(res => {
          console.log('res', res)
          this.model = Object.assign({}, res.result)
          this.pollingTable.dataSource = this.model.standardList
        })
      } else {
        this.model = Object.assign({}, record)
      }
      this.visible = true
    },
    handleDetail(record) {
      console.log('record', record)
      getAction(this.url.queryByEquipmentId + record.id).then(res => {
        console.log('res', res)
        if (res.result.length > 0) {
          let list = res.result
          list.forEach(item => {
            this.SpotTab.dataSource = item.patrolExecutionSecondaryList
          })
          list.forEach(item => {
            this.SpotTab.dataSource = item.patrolExecutionSecondaryList
          })
        }
      })
      console.log('this.SpotTab.dataSource', this.SpotTab.dataSource)
      this.SpotVis = true
    },
    SpotOk() {
      this.$refs.spotRef.getValues((error, values) => {
        this.SpotTab.dataSource = values
        httpAction(this.url.editTask, this.SpotTab.dataSource, 'post').then(res => {
          if (res.success) {
          } else {
            this.$message.warning(res.message)
          }
        })
      })
      this.SpotTab.dataSource = []
      this.SpotVis = false
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
