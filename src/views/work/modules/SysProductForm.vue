<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="产品编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cod">
              <a-input v-model="model.code" placeholder="请输入,忽略将自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="产品名称"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="productName"
              :rules="[{ required: true, message: '请输入产品名称!' }]"
            >
              <a-input v-model="model.productName" placeholder="请输入产品名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="库存单位"
              :rules="disabled ? null : [{ required: true, message: '库存单位不能为空!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="stockPiece"
            >
              <a-select
                defaultValue="alipay"
                @change="fieldsChange(model.stockPiece)"
                placeholder="请选择"
                v-model="model.stockPiece"
                :showSearch="true"
                optionFilterProp="value"
              >
                <a-select-option v-for="(item, index) in stockPieceList" :key="index" :value="item.name"
                  >{{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productNorms">
              <a-input v-model="model.productNorms" placeholder="请输入产品规格"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="产品属性"
              :rules="disabled ? null : [{ required: true, message: '请输入产品属性!' }]"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="productProperty"
            >
              <a-select
                style="width: 100%"
                v-model="model.productProperty"
                placeholder="请输入产品属性"
                :allowClear="true"
              >
                <a-select-option v-for="(action, index) in textList" :key="index" :value="action.value"
                  >{{ action.label }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="工艺路线" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="craftRouteId">
              <a-select
                defaultValue="alipay"
                placeholder="请选择"
                v-model="model.craftRouteId"
                :showSearch="true"
                optionFilterProp="label"
              >
                <a-select-option v-for="(item, index) in LineList" :key="index" :value="item.id" :label="item.name"
                  >{{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="最大库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="maxStock">
              <a-input-number v-model="model.maxStock" placeholder="请输入最大库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="最小库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="minStock">
              <a-input-number v-model="model.minStock" placeholder="请输入最小库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="安全库存" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="securityStock">
              <a-input-number v-model="model.securityStock" placeholder="请输入安全库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="库存数量" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockNumber">
              {{ this.model.stockNumber ? this.model.stockNumber : 0 }}
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="材质" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="material">
              <a-input v-model="model.material" placeholder="请输入安全库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <!-- <a-col :span="12">
            <a-form-model-item label="在制品总数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="securityStock">
              <a-input-number v-model="model.a" placeholder="请输入安全库存" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item
              label="在制品已入库数"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="securityStock"
            >
              <a-input-number v-model="model.c" placeholder="请输入安全库存" style="width: 100%" />
            </a-form-model-item>
          </a-col> -->
          <a-col :span="24">
            <div>
              <a-form-model-item label="关联图纸" :labelCol="{ span: 2 }" :wrapperCol="{ span: 16 }">
                <a-upload
                  style="margin: 20px 0 0 20px;"
                  name="file"
                  listType="picture-card"
                  :action="addFile"
                  :fileList="fileList"
                  :multiple="true"
                  :headers="tokenHeader"
                  :beforeUpload="beforeUploadMains"
                  @change="handleChange"
                  @preview="handlePreview"
                >
                  <div v-if="this.fileList.length < 10">
                    <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
                    <div class="ant-upload-text">上传</div>
                  </div>
                </a-upload>
              </a-form-model-item>
            </div>
          </a-col>
          <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible = false">
            <img alt="example" style="width: 100%" :src="previewImage" />
          </a-modal>
          <!-- 自定义字段区域 -->
          <a-col v-if="model.expandFieldValues" :span="12">
            <a-form-model-item
              v-for="(item, index) in model.expandFieldValues"
              :label="item.name"
              :key="index"
              :rules="[{ required: item.blRequired == '1', message: '请输入!' }]"
              :labelCol="{
                xs: { span: 24 },
                sm: { span: 6 }
              }"
              :wrapperCol="{
                xs: { span: 24 },
                sm: { span: 17 }
              }"
              :prop="item.expands.fieldValue"
            >
              <!-- 
              :prop="'fieldValue.' + item.expands.fieldValue + '.value'"
              :prop="item.expands.fieldValue"
              :name="item.expands.fieldValue"
             -->
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
                    <a-col v-for="(itemx, index) in JSON.parse(item.options)" :key="index" :span="12">
                      <a-checkbox :value="itemx">{{ itemx }}</a-checkbox>
                    </a-col>
                  </a-row>
                </a-checkbox-group>
              </a-tooltip>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { httpAction, getAction } from '@/api/manage'
import { postAction } from '../../../api/manage'

export default {
  name: 'SysProductForm',
  mixins: [JeecgListMixin],
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
      // 关联图纸
      uploadLoading: false,
      headers: {},
      fileList: [],
      fileListId: [],
      previewImage: '',
      previewVisible: false,
      // end
      // 自定义字段
      model: {
        expandFieldValues: null
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 18 }
      },
      confirmLoading: false,
      validatorRules: {
        code: [{ required: true, message: '请输入产品编码!' }]
      },
      url: {
        add: '/longxianda/sysProduct/add',
        edit: '/longxianda/sysProduct/edit',
        addFile: '/longxianda/BaseFileController/addFile',
        queryById: '/longxianda/sysProduct/queryById?id=',
        equipmentList: ''
      },
      stockPieceList: null,
      LineList: null,
      textList: [
        { label: '自制', value: '自制', defaultChecked: true },
        { label: '外购', value: '外购', defaultChecked: false }
      ]
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    },
    addFile: function() {
      return `${window._CONFIG['domianURL']}/${this.url.addFile}`
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.loadStockPieceList()
    this.loadLineList()
  },
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    handleChange(info) {
      let file = info.file
      let fileList = info.fileList
      // console.log('info.file----', info.file)
      // console.log('info.fileList', info.fileList)
      // console.log('info.file.status', info.file.status)
      // 判断number是否大于0和是否多选，返回选定的元素。
      if (file.status === 'done') {
        if (file.response.success) {
          fileList = fileList.map(file => {
            if (file.response) {
              file.url = file.response.message
            }
            return file
          })
        }
      } else if (file.status === 'error') {
        this.$message.warning(`${file.name} 上传失败.`)
      } else if (file.status === 'removed') {
        this.handleDelete(file)
      } else {
        let pm = new FormData()
        pm.append('file', file)
        postAction(this.url.addFile, pm).then(res => {
          if (res.success) {
            this.$message.success('上传成功！')
            this.fileListId.push(res.result)
          } else {
            this.$message.error(res.message)
          }
        })
      }
      this.fileList = fileList
    },
    handleDelete(file) {
      //如有需要新增 删除逻辑
      console.log(file)
    },
    beforeUploadMains(file, e) {
      console.log('file.type', file.type)
      let isJpgOrPng =
        file.type === 'image/jpeg' ||
        file.type === 'image/png' ||
        file.type === 'image/jpg' ||
        file.type === 'application/pdf'
      if (!isJpgOrPng) {
        this.$message.error('格式错误，只能上传jpg、jpeg、png')
        return Promise.reject(false)
      }
      return false // 阻止upload默认上传事件
    },
    // 预览
    handlePreview(file) {
      this.previewImage = file.url || file.thumbUrl
      this.previewVisible = true
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
      if (record.id) {
        this.getInfo(record.id)
      }
      this.visible = true
      if (!this.model.id) {
        this.loadExpandFieldValues(1001)
      }
    },
    // 编辑时,通过id获取详情
    getInfo(id) {
      var that = this
      getAction(this.url.queryById + id)
        .then(res => {
          if (res.success) {
            this.model = Object.assign({}, res.result)
            this.model.baseFileList.forEach(item => {
              let pm = {
                name: item.name,
                uid: item.id,
                url: item.path,
                thumbUrl: item.path,
                status: 'done'
              }
              this.fileList.push(pm)
            })
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取单位列表
    loadStockPieceList() {
      var that = this
      let httpurl = '/longxianda/sysSpec/list'
      let pm = { pageNo: 1, pageSize: 10000 }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.stockPieceList = res.result.records
            console.log('res.result.获取单位列表', res.result.records)
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 获取工艺路线列表
    loadLineList() {
      var that = this
      let httpurl = '/longxianda/sysLine/list'
      let pm = { pageNo: 1, pageSize: 10000 }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.LineList = res.result.records
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
    },
    // 如果是新增,通过字典Id获取自定义字段信息
    loadExpandFieldValues(value) {
      var that = this
      let httpurl = '/longxianda/sysCustom/list?targetType=' + value
      let pm = { pageNo: 1, pageSize: 10000 }
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
    fieldsChange(name) {
      this.stockPieceList.map(item => {
        if (item.name == name) {
          this.model.stockPieceId = item.id
        }
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
      // if (that.model.expandFieldValues != null) {
      //   that.model.expandFieldValues.map(item => {
      //     if (item.type == '复选框' && item.expands.fieldValue) {
      //       item.expands.fieldName = item.fieldName
      //       item.expands.ids = item.id
      //       item.expands.fieldValue = JSON.stringify(item.expands.fieldValue)
      //     }
      //   })
      // }
      console.log('that.model.expandFieldValues', that.model.expandFieldValues)
      // 触发表单验证
      that.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          let i = that.model
          //debugger
          if (!that.model.id) {
            httpurl += that.url.add
            method = 'post'
          } else {
            httpurl += that.url.edit
            method = 'put'
          }
          that.model.maxStock = that.model.maxStock ? that.model.maxStock : 0
          that.model.minStock = that.model.minStock ? that.model.minStock : 0
          that.model.securityStock = that.model.securityStock ? that.model.securityStock : 0
          // console.log(' that.model.securityStock', that.model.securityStock)
          // console.log(' that.model.securityStock', that.model)
          that.model.fileListId = this.fileListId
          httpAction(httpurl, that.model, method)
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
