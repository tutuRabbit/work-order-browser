<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code">
              <a-input v-model="model.code" placeholder="请输入,忽略将自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="产品编码"
              :rules="[{ required: true, message: '请填写产品编码!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="productCode"
            >
              <a-select
                defaultValue="alipay"
                placeholder="请选择"
                v-model="model.productCode"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option
                  v-for="(item, index) in produceCodeList"
                  :key="index"
                  @click="fieldsChange(item)"
                  :value="item.code"
                  >{{ item.code }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <div v-if="model.productCode">
            <a-col :span="12">
              <a-form-model-item label="产品ID" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productId">
                {{ model.productId }}
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productName">
                {{ model.productName }}
              </a-form-model-item>
            </a-col>
            <a-col :span="12">
              <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productUnit">
                {{ model.productUnit }}
              </a-form-model-item>
            </a-col>
          </div>
          <a-col :span="12">
            <a-form-model-item
              label="计划数"
              :rules="[{ required: true, message: '请填写计划数!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="planAmount"
            >
              <a-input-number
                v-model="model.planAmount"
                style="width: 100%"
                @change="planChange()"
                placeholder="请输入计划数"
              ></a-input-number>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="计划开始时间"
              :rules="[{ required: true, message: '请填写计划开始时间!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="planStartTime"
            >
              <j-date
                placeholder="请选择计划开始时间"
                v-model="model.planStartTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
                @change="planChange"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="计划结束时间"
              :rules="[{ required: true, message: '请填写计划结束时间!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="planEndTime"
            >
              <j-date
                placeholder="请选择计划结束时间"
                v-model="model.planEndTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
                @change="EndTimeChange"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remarks">
              <a-input v-model="model.remarks" placeholder="请输入"></a-input>
            </a-form-model-item>
          </a-col>
          <div
            v-if="model.productCode && model.qrcodeUrl"
            style="height: 200px; width: 200px; position: absolute; top: -5%; left: 82%"
          >
            <img :src="model.qrcodeUrl" alt="" style="height: 100%; width: 100%" />
          </div>
          <!-- 自定义字段区域 -->
          <a-col v-if="model.expandFieldValues" :span="24">
            <a-form-model-item
              v-for="(item, index) in model.expandFieldValues"
              :label="item.name"
              :key="index"
              :rules="[{ required: item.blRequired == '1', message: '请输入!' }]"
              :labelCol="{ xs: { span: 20 }, sm: { span: 2 } }"
              :wrapperCol="{ xs: { span: 20 }, sm: { span: 20 } }"
              :prop="item.expands.fieldValue"
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
                  <a-radio v-for="(item, index) in JSON.parse(item.options)" :key="index" :value="item">{{
                    item
                  }}</a-radio>
                </a-radio-group>
                <!-- 复选框 -->
                <a-checkbox-group
                  v-if="item.type == '复选框'"
                  v-model="item.expands.fieldValue"
                  :placeholder="item.tips"
                >
                  <a-row>
                    <a-col v-for="(item, index) in JSON.parse(item.options)" :key="index" :span="8">
                      <a-checkbox :value="item">{{ item }}</a-checkbox>
                    </a-col>
                  </a-row>
                </a-checkbox-group>
              </a-tooltip>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
    <!-- 子表单区域 -->
    <a-col>
      <div style="margin-bottom: 10px; padding: 0px; font-size: 16px;color: rgb(24,144,255);">
        <span>生产任务</span>
        <a-select
          v-if="isAdd == 1"
          style="width: 200px;float: right"
          placeholder="请选择"
          :filter-option="filterManager"
          :showSearch="true"
          optionFilterProp="label"
        >
          <a-select-option
            v-for="(item, index) in ProcedureList"
            :key="index"
            @click="addChange(item)"
            :value="item.name"
            :label="item.name"
            >{{ item.name }}
          </a-select-option>
        </a-select>
        <span @click="addNext()" style="font-size: 14px;float: right;margin-right: 20px;">+从工序中添加</span>
      </div>
      <a-table
        class-name="my-table"
        style="margin-bottom:50px;z-index: 999999"
        :columns="columns"
        :scroll="{ x: true }"
        :data-source="workOrderDetails"
        :pagination="false"
        :customRow="customRow"
        :rowDrag="true"
      >
        <template slot="manufacturingShop" slot-scope="text, record">
          <j-multi-select-tag
            style="width:100%;z-index: 999999"
            v-model="record.manufacturingShop"
            placeholder="请选择"
            dictCode="manufacturing_shop"
          />
        </template>
        <template slot="name" slot-scope="text, record">
          <j-multi-select-tag
            style="width: 100%;z-index: 999999"
            v-model="record.taskOperators"
            placeholder="请选择"
            @change="$forceUpdate()"
            dictCode="sys_user,realname,id"
          />
        </template>
        <template slot="startTime" slot-scope="text, record">
          <a-date-picker
            @change="$forceUpdate()"
            placeholder="请选择计划开始时间"
            v-model="record.planStartTime"
            show-time
            format="YYYY-MM-DD HH:mm:ss"
            valueFormat="YYYY-MM-DD HH:mm:ss"
            style="width: 100%"
          />
        </template>
        <template slot="endTime" slot-scope="text, record">
          <a-date-picker
            @change="$forceUpdate()"
            placeholder="请选择计划结束时间"
            v-model="record.planEndTime"
            show-time
            format="YYYY-MM-DD  HH:mm:ss"
            valueFormat="YYYY-MM-DD  HH:mm:ss"
            style="width: 100%"
          />
        </template>
        <span slot="action" slot-scope="text, record, index">
          <a-popconfirm title="确定删除吗?" @confirm="() => delNext(index)">
            <a>删除</a>
          </a-popconfirm>
        </span>
      </a-table>
    </a-col>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'

export default {
  name: 'SysWorkOrderForm',
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
      columns: [
        {
          title: '工序名称',
          align: 'center',
          dataIndex: 'procedureName'
        },
        {
          title: '工序编号',
          align: 'center',
          dataIndex: 'procedureCode'
        },
        {
          title: '加工参数',
          align: 'center',
          dataIndex: 'machiningParameter'
        },
        {
          title: '报工权限',
          align: 'center',
          dataIndex: 'outputUsers'
        },
        {
          title: '任务分配',
          align: 'center',
          width: 280,
          dataIndex: 'taskOperatorsStr',
          scopedSlots: { customRender: 'name' }
        },
        {
          title: '生产车间',
          align: 'center',
          width: 300,
          dataIndex: 'manufacturingShop',
          scopedSlots: { customRender: 'manufacturingShop' }
        },
        {
          title: '报工数配比',
          align: 'center',
          width: 120,
          dataIndex: 'outputRate'
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount'
        },
        {
          title: '不良品项列表',
          align: 'center',
          dataIndex: 'defectsStr'
        },
        {
          title: '计划开始时间',
          align: 'center',
          width: 200,
          dataIndex: 'planStartTime',
          scopedSlots: { customRender: 'startTime' }
        },
        {
          title: '计划结束时间',
          align: 'center',
          width: 200,
          dataIndex: 'planEndTime',
          scopedSlots: { customRender: 'endTime' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      model: { expandFieldValues: null },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 3 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysWorkOrder/add',
        edit: '/longxianda/sysWorkOrder/edit',
        queryById: '/longxianda/sysWorkOrder/queryById'
      },
      isAdd: 0,
      produceCodeList: null, // 产品列表
      ProcedureList: [], // 工序列表
      workOrderDetails: [], // 生产任务列表
      targetObj: {},
      sourceObj: {}
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
    this.loadProduceCodeList()
    this.loadProcedureList()
  },
  methods: {
    //判断数组中是否有相同的元素
    hasDuplicate(arr) {
      for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length; j++) {
          if (arr[i] === arr[j]) {
            return true
          }
        }
      }
      return false
    },
    startTimeChange(val) {
      this.$forceUpdate()
    },
    filterManager(input, option) {
      // option.componentOptions.children[0].text.toLowerCase()为内容值
      // option.data.attrs.data_mobile为绑定的属性值，可自行定义
      return (
        option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0 ||
        option.data.attrs.data_mobile.indexOf(input.toLowerCase()) >= 0
      )
    },
    // 自定义拖拽
    customRow(record, index) {
      return {
        props: {
          draggable: 'true'
        },
        style: {
          cursor: 'pointer'
        },
        on: {
          // 鼠标移入
          mouseenter: event => {
            // 兼容IE
            var ev = event || window.event
            ev.target.draggable = true
          },
          // 开始拖拽
          dragstart: event => {
            // 兼容IE
            var ev = event || window.event
            // 阻止冒泡
            ev.stopPropagation()
            // 得到源目标数据
            this.sourceObj = record
            console.log(
              '%c [ sourceObj ]-442',
              'font-size:13px; background:pink; color:#bf2c9f;',
              JSON.stringify(this.sourceObj)
            )
          },
          // 拖动元素经过的元素
          dragover: event => {
            // 兼容 IE
            var ev = event || window.event
            // 阻止默认行为
            ev.preventDefault()
          },
          // 鼠标松开
          drop: event => {
            // 兼容IE
            var ev = event || window.event
            // 阻止冒泡
            ev.stopPropagation()
            // 得到目标数据
            this.targetObj = record
            const tempData = this.workOrderDetails

            tempData[this.targetObj.weight] = this.sourceObj
            tempData[this.sourceObj.weight] = this.targetObj
            let weightList = []
            tempData.forEach((item, index) => {
              item.weight = index
              weightList.push(item)
            })
            this.workOrderDetails = weightList //更改顺序
            if (hasDuplicate(weightList)) {
              this.workOrderDetails = [...new Set(weightList)]
              this.workOrderDetails.splice(index, 0, this.sourceObj)
            }
          }
        }
      }
    },

    // end
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
    //点击上移
    clickUp(index) {
      this.swapArray(this.workOrderDetails, index - 1, index)
    },
    //点击下移
    clickDown(index) {
      this.swapArray(this.workOrderDetails, index, index + 1)
    },
    //数组元素互换位置
    swapArray(arr, index1, index2) {
      arr[index1] = arr.splice(index2, 1, arr[index1])[0]
      return arr
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      if (record.id) {
        this.getInfo(record.id)
      }
      this.visible = true
      if (!this.model.id) {
        this.loadExpandFieldValues(1007)
      }
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
    // 时间范围
    onBirthdayChange: function(value, dateString) {
      console.log(dateString[0], dateString[1])
      this.model.planStartTime = dateString[0]
      this.model.planEndTime = dateString[1]
      this.handleScrwList()
    },
    // 产品信息监听
    fieldsChange(info) {
      let that = this
      // 产品ID
      this.model.procedureId = info.id
      // 产品ID
      this.model.productId = info.id
      // 产品名称
      this.model.productName = info.productName
      // 产品规格
      this.model.productUnit = info.productNorms

      this.loadProcedureListById(this.model.productId)
    },
    // 监听时间范围是否合理
    EndTimeChange() {
      let i = Date.parse(this.model.planEndTime) - Date.parse(this.model.planStartTime)
      if (i < 0) {
        this.model.planEndTime = null
      } else {
        this.handleScrwList()
      }
    },
    // 计划监听
    planChange() {
      this.handleScrwList()
    },
    // 处理生产列表
    handleScrwList() {
      var that = this
      if (this.workOrderDetails.length > 0) {
        this.workOrderDetails.map(item => {
          item.planAmount = that.model.planAmount // 计划数
          item.planStartTime = that.model.planStartTime // 计划开始时间
          item.planEndTime = that.model.planEndTime // 计划结束时间
          if (!item.id) {
            item.procedureCode = item.code
            item.procedureName = item.name
            item.outputUsers = item.output
            item.outputRate = item.outputRate
          }
        })
      }
      console.log('this.workOrderDetails', this.workOrderDetails)
    },
    // 获取产品列表
    loadProduceCodeList() {
      var that = this
      let httpurl = '/longxianda/sysProduct/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000,
        order: 'desc'
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
    // 获取工序列表
    loadProcedureList() {
      var that = this
      let httpurl = '/longxianda/sysProcedure/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000,
        order: 'desc'
      }
      getAction(httpurl, pm)
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
    // 通过产品ID获取工序列表
    loadProcedureListById(productId) {
      var that = this
      let httpurl = '/longxianda/sysLine/queryByProduct?productId=' + productId
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            var list = res.result
            if (list.length > 0) {
              list.map(item => {
                item.procedureId = item.id
                delete item.id // 删除id
              })
              console.log('list', list)
              that.workOrderDetails = list
              that.handleScrwList()
            } else {
              that.workOrderDetails = []
            }
          } else {
            that.$message.warning(res.message)
            that.workOrderDetails = []
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 添加子表项
    addNext(e) {
      if (this.isAdd == 0) {
        this.isAdd = 1
      } else {
        this.isAdd = 0
      }
    },
    addChange(info) {
      console.log('info', info)
      info.procedureId = info.id
      delete info.id // 删除id
      this.workOrderDetails.push(info)
      console.log('this.workOrderDetails', JSON.stringify(this.workOrderDetails))
      this.handleScrwList()
      this.isAdd = 0
      this.$forceUpdate()
    },
    // 删除子表项
    delNext(index) {
      this.workOrderDetails.splice(index, 1)
    },
    // 编辑时,通过id获取详情
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysWorkOrder/queryById?id=' + id
      httpAction(httpurl, '', 'get')
        .then(res => {
          if (res.success) {
            this.model = Object.assign({}, res.result)
            // this.model.workOrderDetails.forEach(item => {
            //   delete item.id // 删除id
            //   this.workOrderDetails.push(item)
            // })
            this.workOrderDetails = this.model.workOrderDetails
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
      this.model.expandFieldValues.map(item => {
        if (item.type == '复选框' && item.expands.fieldValue) {
          item.expands.fieldName = item.fieldName
          item.expands.ids = item.id
          item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
        }
      })
      // this.model.status = 0
      this.workOrderDetails.map((item, index) => {
        item.procedureOrder = index + 1
      })
      this.model.workOrderDetails = this.workOrderDetails
      // 触发表单验证
      this.$refs.form.validate(valid => {
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
          if (this.model.workOrderDetails.length <= 0) {
            that.$message.warning('生产任务不能为空!')
            that.confirmLoading = false
            return
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
<style lang="less" scoped>
::v-deep .ant-table-body tr > td {
  width: 200px;
}
::v-deep .ant-select-dropdown {
  z-index: 999999 !important;
}
.my-table tbody > tr:hover > td {
  background-color: unset;
}
</style>
