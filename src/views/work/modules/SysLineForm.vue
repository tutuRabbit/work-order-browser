<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled" :whith="1000">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="工艺路线编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="code">
              <a-input v-model="model.code" placeholder="请输入,忽略将自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item
              label="工艺路线名称"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="name"
              :rules="[{ required: true, message: '请输入工艺路线名称!' }]"
            >
              <a-input v-model="model.name" placeholder="请输入工艺路线名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="工序列表" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fields">
              <a-button type="primary" @click="addNext" style="margin-bottom: 10px">
                添加一行
              </a-button>
              <a-button type="primary" @click="saveForm" style="float:right">
                另存
              </a-button>
              <a-table :columns="columns" :data-source="listCont" :pagination="false" :customRow="customRow">
                <span slot="action" slot-scope="text, record, index">
                  <a-popconfirm title="确定删除吗?" @confirm="() => delNext(index)">
                    <a>删除</a>
                  </a-popconfirm>
                </span>
                <span slot="procedureName" slot-scope="text, record, index">
                  <a-radio-group @change="fieldsChange(index)" v-model="record.type">
                    <a-radio value="工序">工序</a-radio>
                    <a-radio value="工艺线路">工艺线路</a-radio>
                  </a-radio-group>
                </span>
                <span slot="outputUsers" slot-scope="text, record, index">
                  <a-select
                    v-if="record.type == '工序'"
                    defaultValue="alipay"
                    style="width: 150px"
                    v-model="record.codeName"
                    :showSearch="true"
                    optionFilterProp="value"
                  >
                    <a-select-option
                      v-for="(item, indexx) in recordList"
                      :key="item.id"
                      @click="lineInfosChange(item, index, indexx)"
                      :value="item.name"
                      >{{ item.name }}
                    </a-select-option>
                  </a-select>
                  <a-select
                    v-if="record.type == '工艺线路'"
                    defaultValue="alipay"
                    style="width: 150px"
                    v-model="record.codeName"
                    :showSearch="true"
                    optionFilterProp="value"
                  >
                    <a-select-option
                      v-for="(item, indexx) in procedureList"
                      :key="item.id"
                      @click="lineInfosChange(item, index, indexx)"
                      :value="item.name"
                      >{{ item.name }}
                    </a-select-option>
                  </a-select>
                </span>
                <span slot="outputRate" slot-scope="text, record">
                  <a-input v-if="record.type === '工序'" v-model="record.report" placeholder="请填写"></a-input>
                </span>
                <span slot="manufacturingShopStr" slot-scope="text, record">
                  <!-- <span>{{ record.manufacturingShopStr }}</span> -->
                  <a-input readonly v-if="record.type === '工序'" v-model="record.manufacturingShopStr"></a-input>
                </span>
              </a-table>
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
import STable from '@/components/table/'

export default {
  name: 'SysLineForm',
  components: {
    STable
  },
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
        xs: {
          span: 24
        },
        sm: {
          span: 3
        }
      },
      wrapperCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 20
        }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/longxianda/sysLine/add',
        edit: '/longxianda/sysLine/edit',
        queryById: '/longxianda/sysLine/queryById'
      },
      listCont: [],
      recordList: [],
      procedureList: [],
      columns: [
        {
          title: '类型',
          align: 'center',
          width: 200,
          dataIndex: 'procedureName',
          scopedSlots: { customRender: 'procedureName' }
        },
        {
          title: '工序/工艺路线',
          align: 'center',
          width: 250,
          dataIndex: 'outputUsers',
          scopedSlots: { customRender: 'outputUsers' }
        },
        {
          title: '报工数配比',
          align: 'center',
          dataIndex: 'outputRate',
          scopedSlots: { customRender: 'outputRate' }
        },
        {
          title: '工艺参数',
          align: 'center',
          dataIndex: 'machiningParameter'
        },
        {
          title: '生产车间',
          align: 'center',
          width: 120,
          dataIndex: 'manufacturingShopStr',
          scopedSlots: { customRender: 'manufacturingShopStr' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          width: 80,
          scopedSlots: { customRender: 'action' }
        }
      ],
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
  },
  methods: {
    inputChange(val) {
      console.log('val', val)
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
            const tempData = this.listCont
            tempData[this.targetObj.weight] = this.sourceObj
            tempData[this.sourceObj.weight] = this.targetObj
            let weightList = []
            tempData.forEach((item, index) => {
              item.weight = index
              weightList.push(item)
            })
            this.listCont = weightList //更改顺序
            // console.log('weightList', weightList)
          }
        }
      }
    },
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      this.model = Object.assign({}, record)
      if (this.model.id) {
        this.getInfo(this.model.id)
        // this.fieldsChange(1)
        getAction('/longxianda/sysProcedure/list', { pageNo: 1, pageSize: 10000 }).then(res => {
          if (res.success) {
            this.recordList = res.result.records
            console.log(' this.recordList', this.recordList)
          }
        })
        getAction('/longxianda/sysLine/list', { pageNo: 1, pageSize: 10000 }).then(res => {
          if (res.success) {
            this.procedureList = res.result.records
            console.log(' this.procedureList', this.procedureList)
          }
        })
      }
      this.visible = true
    },
    getInfo(id) {
      var that = this
      let httpurl = '/longxianda/sysLine/queryById?id=' + id
      getAction(httpurl, { pageNo: 1, pageSize: 10000 })
        .then(res => {
          if (res.code == 200) {
            that.model = res.result
            that.listCont = that.model.lineInfos
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    numCheck(rule, value, callback, record) {
      if (!record.tsPrizeInfo.prizeNum) {
        callback(new Error(rule.message))
      } else {
        callback()
      }
    },
    fieldsChange(index) {
      console.log('index', index)
      this.listCont[index].codeName = ''
      var that = this
      var listCont = this.listCont
      console.log('listCont[index].type', listCont[index].type)
      let httpurl = ''
      if (listCont[index].type == '工序') {
        httpurl = '/longxianda/sysProcedure/list'
        getAction(httpurl, { pageNo: 1, pageSize: 10000 })
          .then(res => {
            if (res.success) {
              that.recordList = res.result.records
            } else {
              that.$message.warning(res.message)
            }
          })
          .finally(() => {
            that.confirmLoading = false
          })
        this.$message.warning(`请选择工序！！`)
      } else {
        // 工艺线路
        httpurl = '/longxianda/sysLine/list'
        getAction(httpurl, { pageNo: 1, pageSize: 10000 })
          .then(res => {
            if (res.success) {
              that.procedureList = res.result.records
            } else {
              that.$message.warning(res.message)
            }
          })
          .finally(() => {
            that.confirmLoading = false
          })
        this.$message.warning(`请选择工艺线路！！`)
      }
    },
    lineInfosChange(info, index, x) {
      // console.log('index', index)
      // console.log('this.listCont', this.listCont)
      // console.log('info', info)
      // console.log('info.machiningParameter', info.machiningParameter)
      this.listCont[index].code = info.code
      this.listCont[index].procedureId = info.id
      this.listCont[index].report = info.outputRate
      this.listCont[index].machiningParameter = info.machiningParameter
      this.listCont[index].manufacturingShopStr = info.manufacturingShopStr
      this.listCont[index].manufacturingShop = info.manufacturingShop
      this.$forceUpdate()
    },
    addNext(e) {
      var listCont = this.listCont
      var pd = {
        code: '',
        machiningParameter: '',
        manufacturingShopStr: '',
        manufacturingShop: '',
        codeName: '',
        report: 0,
        seq: 0,
        type: null
      }
      listCont.push(pd)
      this.recordList = []
      this.procedureList = []
    },
    delNext(index) {
      var listCont = this.listCont
      listCont.splice(index, 1)
    },
    saveForm() {
      delete this.model.id
      console.log('this.model.id', this.model.id)
      console.log('this.model', this.model)
      this.submitForm()
    },
    submitForm() {
      const that = this
      let num = -1
      console.log(' this.listCont', this.listCont)
      for (let i = 0; i < this.listCont.length; i++) {
        delete this.listCont[i].id
        this.listCont[i].seq = i + 1
        num = i
        if (!this.listCont[i].type) {
          this.$message.warning(`请选择类型！！`)
          return
        }
        if (!this.listCont[i].codeName) {
          this.$message.warning(`请选择工序/工艺线路！！`)
          return
        }
        if (this.listCont[i].type == '工序' && !this.listCont[i].report) {
          this.$message.warning(`请填写报工配比！！`)
          return
        }
      }
      this.model.lineInfos = this.listCont
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
