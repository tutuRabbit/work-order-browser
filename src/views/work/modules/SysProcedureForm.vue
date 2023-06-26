<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="工序编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code">
              <a-input v-model="model.code" placeholder="请输入,忽略将自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="工序名称"
              :rules="[{ required: true, message: '请填写工序名称!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="name"
            >
              <a-input v-model="model.name" placeholder="请填写" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="加工参数"
              :rules="[{ required: true, message: '请填写加工参数!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="machiningParameter"
            >
              <a-input v-model="model.machiningParameter" placeholder="请填写" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="报工权限" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outputIdList">
              <a-tree-select
                v-model="model.outputIdList"
                style="width: 100%"
                :tree-data="permissionList"
                tree-checkable
                :show-checked-strategy="SHOW_PARENT"
                :allowClear="true"
                placeholder="请选择"
                tree-node-filter-prop="title"
                @change="outputChange"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="报工数配比"
              :rules="[{ required: true, message: '请填写报工数配比!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="outputRate"
            >
              <a-input-number v-model="model.outputRate" placeholder="请输入报工数配比" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="不良品项列表" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="birthDefects">
              <a-select
                style="width: 100%"
                mode="multiple"
                placeholder="请选择"
                v-model="model.birthDefects"
                :allowClear="true"
                @change="handleChange"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option v-for="(item, index) in rejectsLsit" :key="index" :label="item.name" :value="item.code"
                  >{{ item.name }}
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
                placeholder="请选择"
                dictCode="manufacturing_shop"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="是否质检" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="qualityInspection">
              <a-radio-group v-model="model.qualityInspection">
                <a-radio value="1">是</a-radio>
                <a-radio value="0">否</a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <!-- 自定义字段区域 -->
          <a-col v-if="model.expandFieldValues" :span="24">
            <a-form-model-item
              v-for="(item, index) in model.expandFieldValues"
              :label="item.name"
              :key="index"
              :rules="[{ required: item.blRequired == '1', message: '请输入!' }]"
              :labelCol="{ span: 2 }"
              :wrapperCol="{ span: 21 }"
              :prop="item.expands.id"
            >
              <!-- <a-form-model-item
              v-for="(item, index) in model.expandFieldValues"
              :label="item.name"
              :key="index"
              :rules="[{ required: item.blRequired == '1', message: '请输入!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              :prop="item.expands.fieldValue"
            > -->
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

          <a-col :span="24">
            <a-form-model-item label="工序采集数据" :labelCol="{ span: 2 }" :wrapperCol="{ span: 21 }" prop="fields">
              <!-- <a-button icon="plus" type="primary" @click="addChange">新增</a-button> -->
              <j-editable-table
                ref="fieldsTable"
                :loading="table.loading"
                :columns="table.columns"
                :dataSource="table.dataSource"
                :rowNumber="true"
                :rowSelection="true"
                :actionButton="true"
                :dragSort="true"
              >
                <template v-slot:action="props">
                  <a @click="handleDelete(props)">删除</a>
                </template>
              </j-editable-table>
              <!-- <j-editable-table
                ref="fieldsTable"
                :loading="table.loading"
                :columns="table.columns"
                :dataSource="table.dataSource"
                :maxHeight="350"
                :rowNumber="true"
                :rowSelection="true"
                :actionButton="false"
              >
                <template v-slot:action="props">
                  <a @click="handleDelete(props)">删除</a>
                </template>
              </j-editable-table> -->
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import JEditableTable from '@/components/jeecg/JEditableTable'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { TreeSelect } from 'ant-design-vue'

const SHOW_PARENT = TreeSelect.SHOW_PARENT

export default {
  name: 'SysProcedureForm',
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
      SHOW_PARENT,
      outputIds: '',
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '类型',
            key: 'working',
            type: FormTypes.select,
            options: [
              { title: '文本', value: '1' },
              { title: '数字', value: '2' },
              { title: '时间', value: '3' },
              { title: '照片', value: '4' }
            ],
            placeholder: '请选择${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '名称',
            key: 'report',
            width: '40%',
            type: FormTypes.input,
            defaultValue: '',
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '操作',
            key: 'action',
            width: '320px',
            type: FormTypes.slot, // 定义该列为 自定义插值列
            slotName: 'action' // slot 的名称，对应 v-slot 冒号后面和等号前面的内容
          }
        ]
      },
      model: {
        outputIdList: [],
        birthDefects: [],
        expandFieldValues: null
      },
      labelCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 4
        }
      },
      wrapperCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 18
        }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysProcedure/add',
        edit: '/longxianda/sysProcedure/edit',
        queryById: '/longxianda/sysProcedure/queryById'
      },
      recordList: [],
      // 不良品项列表
      rejectsLsit: null,
      // 不良品项值列表
      defectsList: [],
      // 向后端拉取可以用的操作列表
      permissionList: [
        {
          title: '所有人',
          value: 'all',
          key: 'all',
          children: []
        }
      ]
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
    // 获取不良品项列表
    this.loadRejectsLsit()
    // 获取报工权限列表
    this.loadPermissionList()
    // 工序采集数据类型列表
    this.fieldsChange()
  },
  methods: {
    handleDelete(props) {
      // 参数解释
      // props.index ：当前行的下标
      // props.text ：当前值，可能是defaultValue定义的值，也可能是从dataSource中取出的值
      // props.rowId ：当前选中行的id，如果是新增行则是临时id
      // props.column ：当前操作的列
      // props.getValue ：这是一个function，执行后可以获取当前行的所有值（禁止在template中使用）
      //                  例：const value = props.getValue()
      // props.target ：触发当前事件的实例，可直接调用该实例内的方法（禁止在template中使用）
      //                  例：target.add()
      // 使用实例：删除当前操作的行
      let { rowId, target } = props
      target.removeRows(rowId)
      console.log('props.index', props.index)
      console.log(' this.table.dataSource', this.table.dataSource)
      // this.table.dataSource.splice(props.index, 1)
    },
    workingChange(val) {
      console.log('val', val)
      this.table.dataSource[val.index].working = val.working
      console.log('this.table.dataSource[val.index]', this.table.dataSource[val.index])
      this.$forceUpdate()
    },
    // 自定义字段（赋值监听）
    fieldValueChange(i) {
      // console.log('this.model.expandFieldValues', this.model.expandFieldValues)
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

    loadRejectsLsit() {
      var that = this
      let httpurl = '/longxianda/sysRejects/list'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.rejectsLsit = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    loadPermissionList() {
      var that = this
      let httpurl = '/sys/user/queryOrgOrUserTree'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            this.permissionList[0].children = res.result
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      this.model = Object.assign({}, record)
      if (this.model.id) {
        let list = []
        this.model.defects = JSON.parse(record.defects)
        this.model.defects.map(item => {
          list.push(item.code)
        })

        this.model.defects = list
        this.model.birthDefects = list
        this.defectsList = list
        this.table.dataSource = JSON.parse(record.fields) || [] // 工序采集数据
        if (record.outputBlAll == 1) {
          this.outputIds = 'all'
          this.model.outputIdList = 'all'
        } else {
          this.outputIds = JSON.parse(record.outputIds) // 报工权限
          this.model.outputIdList = JSON.parse(record.outputIds) // 报工权限
        }
      }
      this.visible = true
      if (!this.model.id) {
        this.model.qualityInspection = '0'
        this.model.outputRate = 0
        this.loadExpandFieldValues(1002)
      }
      // console.log('  this.model.expandFieldValues', this.model.expandFieldValues)
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
            console.log(' res.result', res.result)
            console.log('res.result.records', res.result.records)
            this.$set(this.model, 'expandFieldValues', res.result.records)
            console.log('that.model.expandFieldValues', that.model.expandFieldValues)
            // that.model.expandFieldValues = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    fieldsChange() {
      var that = this
      let httpurl = '/sys/api/queryEnableDictItemsByCode?code=procedure_collect_date_type'
      let pm = {
        pageNo: 1,
        pageSize: 10000
      }
      getAction(httpurl, pm)
        .then(res => {
          if (res.length > 0) {
            // console.log('res', res)
            // that.recordList = res.result.records
            // that.recordList = res
            // console.log('    that.recordList ', that.recordList)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    outputChange(value) {
      console.log('this.permissionList', this.permissionList)
      this.outputIds = value
      this.model.outputIdList = value
      this.$forceUpdate()
    },
    handleChange(value) {
      this.model.birthDefects = value
      this.model.defects = value
      this.defectsList = value
      this.$forceUpdate()
    },
    addChange() {
      var pd = {
        working: null,
        report: null,
        istype: 'true'
      }
      this.table.dataSource.push(pd)
      console.log('this.table.dataSource', this.table.dataSource)
    },
    delNext(index) {
      // var listCont = this.listCont
      // listCont.splice(index, 1)
    },
    getTreeIds(arr, key) {
      arr.forEach(item => {
        if (key == item.key) {
          if (item.userOrDeptType == '1') {
            // 部门id
            if (this.model.outputDeptIds) {
              this.model.outputDeptIds += key + ','
            } else {
              this.model.outputDeptIds = key + ','
            }
          } else if (item.userOrDeptType == '2') {
            // 个人id
            if (this.model.outputUserIds) {
              this.model.outputUserIds += key + ','
            } else {
              this.model.outputUserIds = key + ','
            }
          }
        } else {
          this.getTreeIds(item.children, key)
        }
      })
    },
    submitForm() {
      const that = this
      // 自定义字段（赋值监听）
      this.model.expandFieldValues.map(item => {
        if (item.type == '复选框' && item.expands.fieldValue) {
          item.expands.fieldName = item.fieldName
          item.expands.ids = item.id
          item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
        }
      })
      // let tableData = that.$refs.fieldsTable.getTableData()
      // let deleteData = that.$refs.fieldsTable.getDeleteData()
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          if (this.model.outputIdList.length == 0) {
            that.$message.warning('请填写报工权限!')
          } else if (this.model.birthDefects.length == 0) {
            that.$message.warning('不良品项不能为空!')
          } else {
            this.$refs.fieldsTable.getValues((error, values) => {
              if (error === 0) {
                that.table.dataSource = values
                // 工序采集数据转json
                this.model.fields = JSON.stringify(this.table.dataSource)
                var rejectsLsit = that.rejectsLsit
                var list = []
                for (var pd of rejectsLsit) {
                  var index = that.defectsList.findIndex(code => code == pd.code)
                  if (index != -1) {
                    list.push({
                      code: pd.code,
                      name: pd.name
                    })
                  }
                }
                // 不良品项列表转json
                this.model.defects = JSON.stringify(list)
                // 报工权限处理
                if (this.outputIds == 'all') {
                  this.model.outputBlAll = 1
                } else {
                  // 报工权限列表转json
                  this.model.outputDeptIds = ''
                  this.model.outputUserIds = ''
                  if (this.model.outputIdList.length != 0 || this.model.outputIdList.typeof == Array) {
                    this.model.outputIdList.forEach(v => {
                      this.getTreeIds(this.permissionList[0].children, v)
                    })
                  }
                  this.model.outputBlAll = 0
                }
                this.model.outputIds = JSON.stringify(this.outputIds)
                that.confirmLoading = true
                let httpurl = ''
                let method = ''
                let s = this.model
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
              } else {
                this.$message.warning('请选填工序采集数据！')
              }
            })
          }
        }
      })
    }
  }
}
</script>
<style lang="less" scoped></style>
