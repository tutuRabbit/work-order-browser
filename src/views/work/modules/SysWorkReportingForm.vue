<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item
              label="工单编号"
              :rules="[{ required: true, message: '请填写工单编号!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="projectCode"
            >
              <a-select
                defaultValue="请选择"
                placeholder="请选择"
                v-model="model.projectCode"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option
                  v-for="(item, index) in produceCodeList"
                  :key="index"
                  @click="itemChange(item)"
                  :value="item.code"
                  >{{ item.code }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <div v-if="model.projectCode">
            <a-col :span="12">
              <a-form-model-item
                label="工序名称"
                :rules="[{ required: true, message: '请填写工序名称!' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="processName"
              >
                <a-select
                  defaultValue="alipay"
                  placeholder="请选择"
                  v-model="model.processName"
                  :showSearch="true"
                  optionFilterProp="value"
                >
                  <a-select-option
                    v-for="(item, index) in ProcedureList"
                    :key="index"
                    @click="ProcedureChange(item)"
                    :value="item.procedureName"
                    >{{ item.procedureName }}
                  </a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="工序状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="processStatus">
                <a-radio-group v-model="model.processStatus" @change="processChange">
                  <a-radio :value="10">执行中</a-radio>
                  <a-radio :value="100">已结束</a-radio>
                </a-radio-group>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item
                label="生产人名称"
                :rules="[{ required: true, message: '请填写生产人!' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="operatorName"
              >
                <a-select
                  defaultValue="请选择"
                  placeholder="请选择"
                  v-model="model.operatorName"
                  :showSearch="true"
                  optionFilterProp="value"
                >
                  <a-select-option
                    v-for="(item, index) in ProducerList"
                    :key="index"
                    @click="ProducerChange(item)"
                    :value="item.realname"
                    >{{ item.realname }}
                  </a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item
                label="生产车间"
                :rules="[{ required: true, message: '请选择生产车间!' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="manufacturingShop"
              >
                <j-multi-select-tag
                  v-model="model.manufacturingShop"
                  disabled
                  placeholder="-"
                  dictCode="manufacturing_shop"
                />
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="产品编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productCode">
                <a-input disabled v-model="model.productCode" placeholder="-"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName">
                <a-input disabled v-model="model.productName" placeholder="-"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productSpec">
                <a-input disabled v-model="model.productSpec" placeholder="-"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planStartTime">
                <j-date
                  placeholder="请选择开始时间"
                  v-model="model.planStartTime"
                  :show-time="true"
                  date-format="YYYY-MM-DD HH:mm:ss"
                  style="width: 100%"
                  @change="$forceUpdate()"
                />
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planEndTime">
                <j-date
                  placeholder="请选择结束时间"
                  v-model="model.planEndTime"
                  :show-time="true"
                  @change="$forceUpdate()"
                  date-format="YYYY-MM-DD HH:mm:ss"
                  style="width: 100%"
                />
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item
                label="报工数"
                :rules="[{ required: true, message: '请填写报工数！' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="outputNumber"
              >
                <a-input-number
                  @change="outputChange"
                  style="width: 50%"
                  v-model="model.outputNumber"
                  placeholder="请输入"
                ></a-input-number>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item
                label="良品数"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="fineNumber"
                v-has="'fineNumber'"
              >
                <a-input-number
                  style="width: 50%"
                  v-model="model.fineNumber"
                  @change="pricingTypeChange()"
                  placeholder="请输入"
                ></a-input-number>
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="工序进度" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fineNumber">
                {{ this.kebaoStr }}
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item
                label="不良品数"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="defectsNumber"
                v-has="'defectsNumber'"
              >
                <a-input-number style="width: 50%" v-model="model.defectsNumber" placeholder="请输入"></a-input-number>
              </a-form-model-item>
            </a-col>
            <a-col v-if="model.defectsNumber" :span="24">
              <a-form-model-item label="不良品项" :labelCol="{ span: '2' }" :wrapperCol="{ span: '21' }" prop="defects">
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
                              <th class="ant-table-cell">名称</th>
                              <th class="ant-table-cell">数量</th>
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
                                <a-select defaultValue="alipay" style="width:100%" v-model="item.defectName">
                                  <a-select-option
                                    v-for="(itemx, indexx) in defectsList"
                                    :key="indexx"
                                    @click="rejectsChange(itemx, index)"
                                    :value="itemx.name"
                                    >{{ itemx.name }}
                                  </a-select-option>
                                </a-select>
                              </td>
                              <td class="ant-table-cell">
                                <a-input-number
                                  style="width:100%"
                                  v-model="item.amount"
                                  placeholder="请填写"
                                ></a-input-number>
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

            <a-col :span="12" v-if="model.pricingType == 1">
              <a-form-model-item
                label="报工时长(单位：分钟)"
                :rules="[{ required: true, message: '请填写报工时长!' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="outputDuration"
              >
                <a-input
                  v-model="model.outputDuration"
                  @change="pricingTypeChange()"
                  placeholder="请输入报工时长"
                ></a-input>
              </a-form-model-item>
            </a-col>
            <div v-has="'performance_information'">
              <a-col :span="24">
                <div class="title">绩效信息</div>
              </a-col>
              <a-col :span="12">
                <a-form-model-item label="计价类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="pricingType">
                  <a-radio-group v-model="model.pricingType" :disabled="true" @change="pricingTypeChange()">
                    <a-radio :value="1">计时</a-radio>
                    <a-radio :value="2">计件</a-radio>
                  </a-radio-group>
                </a-form-model-item>
              </a-col>
              <a-col :span="12">
                <a-form-model-item label="工资单价" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="unitPrice">
                  <a-input disabled v-model="model.unitPrice" placeholder="-"></a-input>
                </a-form-model-item>
              </a-col>
              <a-col :span="12">
                <a-form-model-item label="预计工资" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="expectedSalary">
                  <a-input disabled v-model="model.expectedSalary" placeholder="-"></a-input>
                </a-form-model-item>
              </a-col>
              <a-col :span="12">
                <a-form-model-item label="审批状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="approveStatus">
                  <a-radio-group v-model="model.approveStatus" @change="statusChange">
                    <a-radio :value="0">未审批</a-radio>
                    <a-radio :value="1">已审批</a-radio>
                  </a-radio-group>
                </a-form-model-item>
              </a-col>
              <a-col :span="12">
                <a-form-model-item label="审批人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="approve">
                  <j-dict-select-tag
                    :disabled="true"
                    v-model="model.approve"
                    placeholder="请选择审批人"
                    @change="$forceUpdate()"
                    dictCode="sys_user,realname,id"
                  />
                </a-form-model-item>
              </a-col>
            </div>
            <!-- 自定义字段区域 -->
            <a-col v-if="model.expandFieldValues" :span="24">
              <a-form-model-item
                v-for="(item, index) in model.expandFieldValues"
                :label="item.name"
                :key="index"
                :rules="[{ required: item.blRequired == '1', message: '请输入产品属性!' }]"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                :prop="fieldValue"
              >
                <a-tooltip placement="topLeft">
                  <!-- 字段提示 -->
                  <template v-if="item.description" slot="title">
                    <span>{{ item.description }}</span>
                  </template>
                  <!-- 字段类型 -->
                  <!-- 文本 -->
                  <a-input
                    v-if="item.type == '文本'"
                    v-model="item.expands.fieldValue"
                    :placeholder="item.tips"
                    style="width: 100%"
                    :defaultValue="item.defaultValue"
                    @change="fieldValueChange(index)"
                  />
                  <!-- 大文本 -->
                  <a-input
                    v-if="item.type == '大文本'"
                    v-model="item.expands.fieldValue"
                    type="textarea"
                    :placeholder="item.tips"
                    style="width: 100%"
                    :defaultValue="item.defaultValue"
                    @change="fieldValueChange(index)"
                  />
                  <!-- 数字 -->
                  <a-input-number
                    v-if="item.type == '数字'"
                    v-model="item.expands.fieldValue"
                    :placeholder="item.tips"
                    style="width: 100%"
                    :defaultValue="item.defaultValue"
                    @change="fieldValueChange(index)"
                  />
                  <!-- 时间 -->
                  <a-date-picker
                    v-if="item.type == '时间'"
                    v-model="item.expands.fieldValue"
                    showTime
                    :placeholder="item.tips"
                    valueFormat="YYYY-MM-DD HH:mm:ss"
                    style="width: 100%"
                    @change="fieldValueChange(index)"
                  />
                  <!-- 单选框 -->
                  <a-radio-group
                    v-if="item.type == '单选框'"
                    v-model="item.expands.fieldValue"
                    @change="fieldValueChange(index)"
                    :placeholder="item.tips"
                  >
                    <a-radio v-for="(itemx, index) in JSON.parse(item.options)" :key="index" :value="itemx"
                      >{{ itemx }}
                    </a-radio>
                  </a-radio-group>
                  <!-- 复选框 -->
                  <a-checkbox-group
                    v-if="item.type == '复选框'"
                    v-model="item.expands.fieldValue"
                    :placeholder="item.tips"
                  >
                    <a-row>
                      <a-col v-for="(itemx, indexx) in JSON.parse(item.options)" :key="indexx" :span="8">
                        <a-checkbox :value="itemx">{{ itemx }}</a-checkbox>
                      </a-col>
                    </a-row>
                  </a-checkbox-group>
                </a-tooltip>
              </a-form-model-item>
            </a-col>
          </div>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'
import store from '@/store/'

export default {
  name: 'SysWorkReportingForm',
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
      outputBool: false,
      model: { expandFieldValues: null },
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
        add: '/longxianda/sysWorkReporting/save',
        edit: '/longxianda/sysWorkReporting/edit',
        queryById: '/longxianda/sysWorkReporting/queryById'
      },
      listCont: [],
      produceCodeList: null, // 工单列表
      ProducerList: null, // 生产人列表
      ProcedureList: null, // 工序明细列表
      defectsList: null, // 不良品项列表
      defectsLists: [], // 不良品值列表
      planAmount: 0,
      outputRate: 0,
      fineAmount: 0,
      kebaoStr: '-'
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
    this.loadProduceCodeList(10) // 获取状态为10的工单列表
  },
  methods: {
    processChange() {
      if (!this.model.id) {
        this.model.approveStatus = 0
      }
      this.$forceUpdate()
    },
    statusChange() {
      if (this.model.approveStatus == 0) {
        this.model.approve = ''
      } else {
        this.model.approve = store.getters.userInfo.id
      }
      this.$forceUpdate()
    },
    // 自定义字段（赋值监听）
    fieldValueChange(i) {
      this.model.expandFieldValues.map((item, index) => {
        if (i == index) {
          item.expands.fieldName = item.fieldName
          item.expands.ids = item.id
          if (item.type == '数字') {
            item.expands.fieldValue += ''
          }
          if (item.type == '复选框') {
            item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
          }
        }
      })
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      if (this.model.id) {
        this.outputBool = true
        this.getInfo(record.id)
      } else {
        this.loadExpandFieldValues(2003)
      }
      this.visible = true
    },
    // 如果是新增,通过字典Id获取自定义字段信息
    loadExpandFieldValues(value) {
      var that = this
      let httpurl = '/longxianda/sysCustom/list?targetType=' + value
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.model.expandFieldValues = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 工单响应监听
    itemChange(info) {
      this.model = {}
      let obj = info
      // 获取工序任务列表
      this.loadProcedureList(obj.id)

      this.model.productCode = obj.productCode
      this.model.productName = obj.productName
      this.model.productSpec = obj.productUnit
      this.model.planStartTime = obj.planStartTime
      this.model.planEndTime = obj.planEndTime
      this.model.projectId = obj.id
    },
    // 工序任务响应监听
    ProcedureChange(info) {
      console.log('info', info)
      // 获取生产人列表
      // this.loadProducerList(info.procedureId)
      // 工单code                     工序code
      this.loadProducerList(info.projectCode, info.procedureCode)
      // 通过工序id获取工序信息,得到工序不良品项
      this.loadDefectsList(info.procedureId)
      // 获取绩效信息
      this.loadMeritPayList(info.productCode, info.procedureCode)

      if (!this.model.processStatus) {
        this.model.processStatus = info.procedureStatus // 工序状态
      }
      if (!this.model.id) {
        this.model.manufacturingShop = info.manufacturingShop // 生产车间
      }
      this.model.processCode = info.procedureCode // 工序编号
      this.model.taskListId = info.id // 工序任务ID
      this.planAmount = info.planAmount // 计划数
      this.outputRate = info.outputRate // 报工配比
      this.fineAmount = info.fineAmount // 实际数
      this.kebaoStr = info.fineAmount + '/' + info.planAmount
      //                    实际进度                    总进度
      // this.model.approve = store.getters.userInfo.id
      this.$forceUpdate()
    },
    // 生产人响应监听
    ProducerChange(info) {
      this.model.operatorId = info.id // 生产人ID
    },
    // 报工数change事件
    outputChange(val) {
      console.log('val', val)
      console.log('this.outputBool', this.outputBool)
      console.log('this.outputRate', this.outputRate)
      if (this.outputRate == 0) {
      } else {
        if (!this.outputBool) {
          //报工数= 计划数*报工配比-实际数
          let num1 = this.planAmount * this.outputRate - this.fineAmount
          if (val > num1) {
            this.model.outputNumber = null
            this.$message.warning('报工数不能大于可报工数！')
          }
        } else {
          if (val <= this.planAmount) {
          } else {
            this.$message.warning('报工数不能大于总报工数！')
            this.model.outputNumber = null
          }
        }
      }
      if (this.model.pricingType == 2) {
        // 预计工资计算方式=单价*报工数
        this.model.expectedSalary = this.model.unitPrice * this.model.outputNumber
      }
    },
    // 不良品数、良品数监听  （停用）
    defectsNumberChange() {
      // 报工数等于良品数与不良品数之和
      this.model.outputNumber = this.model.fineNumber

      if (this.model.defectsNumber) {
        this.model.outputNumber = this.model.defectsNumber + this.model.fineNumber
      }
    },
    // 不良品项列表监听
    rejectsChange(info, index) {
      this.listCont[index].defectCode = info.code
    },
    addNext(e) {
      var listCont = this.listCont
      var pd = {
        defectName: '',
        amount: '',
        defectCode: ''
      }
      listCont.push(pd)
    },
    delNext(index) {
      var listCont = this.listCont
      listCont.splice(index, 1)
    },
    // 监听  计件方式
    pricingTypeChange() {
      if (this.model.pricingType == 0) {
        // 计件
        // 计件：单价乘以良品数
        this.model.expectedSalary = this.model.unitPrice * this.model.fineNumber
      } else if (this.model.pricingType == 1) {
        // 计时：单价乘以报工时长
        this.model.expectedSalary = this.model.unitPrice * this.model.outputDuration
      }
      this.$forceUpdate()
    },
    // 获取工单列表
    loadProduceCodeList(status) {
      var that = this
      // let httpurl = '/longxianda/sysWorkOrder/list?status=' + status + '&pageNo=1&pageSize=10000'
      let httpurl = '/longxianda/sysWorkOrder/list?status=' + status
      let pm = {
        pageNo: 1,
        pageSize: 10000,
        order: 'desc',
        column: 'createTime'
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.produceCodeList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取工序任务列表
    loadProcedureList(id) {
      var that = this
      let httpurl = '/longxianda/sysTaskList/list?projectId=' + id + '&pageNo=1&pageSize=999999'
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            that.ProcedureList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取生产人列表
    loadProducerList(projectCode, procedureCode) {
      var that = this
      let pm = {
        pageNo: 1,
        pageSize: 10000,
        projectCode: projectCode,
        procedureCode: procedureCode
      }
      let httpurl = '/longxianda/sysProcedure/queryUsersByProcedure'

      getAction(httpurl, pm)
        .then(res => {
          if (res) {
            that.ProducerList = res
            this.$set(this.model, 'operatorName', store.getters.userInfo.realname)
            // this.model.operatorName = store.getters.userInfo.id
            // if (that.ProducerLis.taskOperators) {
            //   let taskOperators = that.ProducerLis.taskOperators.map(res => res.taskOperators.split(','))
            //   console.log('taskOperators---------------', taskOperators)
            //   let list = res
            //   let arr = list.forEach(item => {
            //     console.log('item', item)
            //   })

            // } else {
            //   that.ProducerList = res
            // }
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 通过工序id,获取工序列表,得到工序不良品项
    loadDefectsList(id) {
      var that = this
      let httpurl = '/longxianda/sysProcedure/queryById?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.defectsList = JSON.parse(res.result.defects)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 通过工序任务id,获取工序任务详情
    loadProcedureInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysTaskList/queryById?id=' + id
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.ProcedureChange(res.result)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取绩效信息
    loadMeritPayList(productCode, procedureCode) {
      // procedureCode 工序
      // productCode 产品
      var that = this
      let httpurl = '/longxianda/sysMeritPay/list?produceCode=' + productCode + '&processCode=' + procedureCode
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.code == 500) {
            that.$message.warning('该产品下的工序暂无绩效工资配置！！')
          } else {
            if (res.result.records.length != 0) {
              let Minfo = res.result.records[0]
              that.$set(that.model, 'pricingType', Minfo.valuationMothod) //改变属性值无响应式使用$set
              that.model.outputId = Minfo.id
              that.model.unitPrice = Minfo.unitPrice
            } else {
              that.$message.warning(res.message)
            }
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 编辑时,通过id获取详情
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysWorkReporting/queryById?id=' + id
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            that.model = Object.assign({}, res.result)
            that.listCont = res.result.defects
            // 预计工资
            if (this.model.pricingType == 2) {
              // 预计工资计算方式=单价*报工数
              this.model.expectedSalary = this.model.unitPrice * this.model.outputNumber
            }
            // 获取工序任务列表
            that.loadProcedureList(res.result.projectId)
            // 获取工序任务信息
            that.loadProcedureInfo(res.result.taskListId)
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
      if (this.model.processStatus == 0) {
        this.model.processStatus = 10
      }
      if (this.model.expandFieldValues != null) {
        this.model.expandFieldValues.map(item => {
          if (item.type == '复选框' && item.expands.fieldValue) {
            item.expands.fieldName = item.fieldName
            item.expands.ids = item.id
            item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
          }
        })
      }
      // 报工时长转小数类型
      this.model.outputDuration = parseFloat(this.model.outputDuration)
      // 不良品项列表转json
      this.model.defects = this.listCont
      // 触发表单验证
      this.$refs.form.validate(valid => {
        //s= 计划数*报工配比-实际数
        let s = this.planAmount * this.outputRate - this.fineAmount
        // outputNumber:报工数;defectsNumber:不良品数;fineNumber:良品数
        if (this.model.outputNumber + this.model.defectsNumber + this.model.fineNumber == 0) {
          that.$message.warning('报工数、良品数、不良品数不能都为0')
        }
        //  else if (this.model.outputNumber > s) {
        //   that.$message.warning('报工数不能大于可报工数！')
        // }
        else {
          if (valid) {
            that.confirmLoading = true
            let httpurl = ''
            let method = ''
            let i = this.model
            if (!this.model.id) {
              httpurl += this.url.add
              method = 'post'
            } else {
              httpurl += this.url.edit
              method = 'post'
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
        }
      })
    }
  }
}
</script>
