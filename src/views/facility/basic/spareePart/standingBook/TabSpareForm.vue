<template>
  <a-spin :spinning="confirmLoading">
    <!-- <j-form-container :disabled="formDisabled"> -->
    <a-form-model ref="form" :model="model" :rules="validatorRules" :disabled="formDisabled">
      <!--  设备信息-->
      <a-row>
        <a-col :span="24">
          <div class="title">设备信息</div>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备件编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareCode">
            <a-input :disabled="formDisabled" v-model="model.spareCode" placeholder="不填可自动生成"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备件名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareName">
            <a-input :disabled="formDisabled" v-model="model.spareName" placeholder="请输入备件名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备件类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareType">
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.spareType"
              dictCode="spare_type"
              placeholder="请选择备件类型"
            >
            </j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareModel">
            <a-input :disabled="formDisabled" v-model="model.spareModel" placeholder="请输入规格型号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="生产厂商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareFirm">
            <a-select
              :disabled="formDisabled"
              v-model="model.spareFirm"
              placeholder="回车可搜索"
              style="width: 100%"
              :showSearch="true"
              optionFilterProp="label"
            >
              <a-select-option
                v-for="(item, index) in spareSupplierOptions"
                :key="index"
                :value="item.id"
                :label="item.companyName"
              >
                {{ item.companyName }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="供应商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareSupplier">
            <a-select
              :disabled="formDisabled"
              :showSearch="true"
              v-model="model.spareSupplier"
              placeholder="回车可搜索"
              style="width: 100%"
              optionFilterProp="label"
            >
              <a-select-option
                v-for="(item, index) in spareSupplierOptions"
                :key="index"
                :value="item.id"
                :label="item.companyName"
              >
                {{ item.companyName }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="更换周期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareCycle">
            <a-input
              :disabled="formDisabled"
              v-model="model.spareCycle"
              style="width: 55%;margin-right:10px"
              placeholder="请输入更换周期"
            ></a-input>
            <j-dict-select-tag
              :disabled="formDisabled"
              style="width: 40%"
              v-model="model.timeType"
              dictCode="time_type"
              placeholder="类型"
            >
            </j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="参考价" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sparePrice">
            <a-input-number
              :disabled="formDisabled"
              v-model="model.sparePrice"
              placeholder="请输入参考价"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="计量单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="spareCountUnit">
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.spareCountUnit"
              dictCode="count_unit"
              placeholder="请选择计量单位"
            >
            </j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="换算单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="converUnit">
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.converUnit"
              dictCode="count_unit"
              placeholder="请选择换算单位"
            >
            </j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="换算比例" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="converScale">
            <a-input :disabled="formDisabled" v-model="model.converScale" placeholder="请输入换算比例"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="库存上限" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockUp">
            <a-input-number
              :disabled="formDisabled"
              @change="checkNum(model.stockUp, 'up')"
              v-model="model.stockUp"
              placeholder="请输入库存上限"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="库存下限" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockLow">
            <a-input-number
              :disabled="formDisabled"
              v-model="model.stockLow"
              placeholder="请输入库存下限"
              @change="checkNum(model.stockLow, 'low')"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input :disabled="formDisabled" v-model="model.remark" placeholder="请输入备注"></a-input>
          </a-form-model-item>
        </a-col>
      </a-row>
      <!-- 关联文档 -->
      <a-row v-if="!disEdit">
        <a-col :span="24">
          <div class="title">关联文档</div>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="选择文件"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <a-upload
              name="file"
              :multiple="true"
              :fileList="fileList"
              :remove="handleRemove"
              :beforeUpload="beforeUpload"
            >
              <a-button :disabled="formDisabled" type="info" icon="upload">选择文件</a-button>
            </a-upload>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="上传提示"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <span>{{ queryParam.title }}</span>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item
            label="文档类型"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 2 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 6 }
            }"
          >
            <j-dict-select-tag
              :disabled="true"
              style="width:100%"
              type="list"
              v-model="queryParam.docType"
              dictCode="doc_type"
              placeholder="请选择文档类型"
            />
          </a-form-item>
        </a-col>
      </a-row>
      <!--关联图片  -->
      <a-row v-if="!disEdit">
        <a-col :span="24">
          <div class="title">关联图片</div>
        </a-col>
        <a-col :span="24">
          <a-upload
            style="margin: 20px 0 0 80px;"
            list-type="picture-card"
            :file-list="pictureFileList"
            :beforeUpload="beforeUploadMains"
            @change="handleChangeImg"
            :showUploadList="{
              showPreviewIcon: false
            }"
          >
            <div v-if="this.pictureFileList.length < 1">
              <!-- 确定上传图片数量 -->
              <a-icon type="plus" />
              <div class="ant-upload-text">上传</div>
            </div>
            <!-- <a-modal :visible="previewVisible" :footer="null" @cancel="previewVisible = false">
              <img alt="example" style="width: 100%" :src="previewImage" />
            </a-modal> -->
          </a-upload>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- </j-form-container> -->
  </a-spin>
</template>

<script>
import { httpAction, getAction, postAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

export default {
  name: 'TabSpareForm',
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
      fileList: [],
      pictureFileList: [],
      linkedId: '',
      disEdit: false,
      model: {
        timeType: 'time1'
      },
      spareSupplierOptions: [],
      queryParam: {
        docType: '2'
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 15 }
      },
      confirmLoading: false,
      validatorRules: {
        spareName: [{ required: true, message: '请输入备件名称' }]
      },
      previewImage: '',
      previewVisible: false,
      url: {
        add: '/spare/tabSpare/add',
        edit: '/spare/tabSpare/edit',
        list: '/spare/tabSpare/queryById',
        queryById: '/spare/tabSpare/queryById',
        addPartPicBind: '/attachment/tabAttachment/addPartPicBind', //添加图片
        addPartDocBind: '/attachment/tabAttachment/addPartDocBind' //添加文档
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    },
    addPartPicBind: function() {
      return `${window._CONFIG['domianURL']}/${this.url.addPartPicBind}`
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    // 生产商、供应商
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      // getAction('/equipment/tabContactCompany/list').then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        console.log('res.供应商', res.result)
        this.spareSupplierOptions = arr
      }
    })
  },
  methods: {
    checkNum(val, type) {
      console.log('type', type)
      if (type == 'up') {
        this.model.stockUp = val >= 0 ? val : 0
        if (this.model.stockLow > this.model.stockUp) {
          this.$message.warning('库存上限请大于库存下限')
          this.model.stockUp = ''
        }
      } else {
        this.model.stockLow = val >= 0 ? val : 0
        if (this.model.stockLow > this.model.stockUp) {
          this.$message.warning('库存下限请小于库存上限')
          this.model.stockLow = ''
        }
      }
    },
    beforeUploadMains(file, e) {
      let isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png' || file.type === 'image/jpg'
      if (!isJpgOrPng) {
        this.$message.error('格式错误，只能上传jpg、jpeg、png')
        return reject(false)
      }
      return false // 阻止upload默认上传事件
    },
    handleChangeImg({ fileList }) {
      this.pictureFileList = fileList
    },
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += sizeList[i] / 1024 / 1024
          // sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'
        if (sum > 30) {
          this.$message.warning('文件总大小超过30MB，请修改后重新上传!')
        }
      }
      return false
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    add() {
      // this.edit(this.modelDefault)
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      getAction('/spare/tabSpare/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.model = res.result
        }
      })
      // this.model = Object.assign({}, record)
      this.disEdit = true
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          // that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          httpAction(httpurl, this.model, method).then(res => {
            if (res.success) {
              if (res.result) {
                this.queryParam.linkedId = res.result
              }
              that.$message.success(res.message)
              that.$emit('ok')
            } else {
              that.$message.warning(res.message)
            }
          })
          // .finally(() => {
          //   that.confirmLoading = false
          // })
          setTimeout(() => {
            // 关联文档
            if (this.fileList.length != 0) {
              var pm = new FormData()
              if (this.fileList.length != 0) {
                let { fileList } = this
                fileList.forEach(file => {
                  pm.append('files', file)
                })
              }
              pm.append('docType', this.queryParam.docType)
              pm.append('linkedId', this.queryParam.linkedId)
              postAction(this.url.addPartDocBind, pm)
                .then(res => {
                  if (res.success) {
                    that.$message.success(res.message)
                    setTimeout(() => {
                      that.$emit('ok')
                    }, 200)
                  } else {
                    that.$message.warning(res.message)
                  }
                })
                .finally(() => {
                  that.confirmLoading = false
                })
            }
            //关联图片
            console.log('this.pictureFileList', this.pictureFileList)
            if (this.pictureFileList.length != 0) {
              var pm = new FormData()
              if (this.pictureFileList.length > 0) {
                this.pictureFileList.forEach(v => {
                  pm.append('pictures', v.originFileObj)
                })
              }
              pm.append('linkedId', this.queryParam.linkedId)
              postAction(this.url.addPartPicBind, pm)
                .then(res => {
                  if (res.success) {
                    that.$message.success(res.message)
                    setTimeout(() => {
                      that.$emit('ok')
                    }, 200)
                  } else {
                    that.$message.warning(res.message)
                  }
                })
                .finally(() => {
                  that.confirmLoading = false
                })
            }
          }, 500)
          if (this.fileList.length == 0) {
            that.$emit('ok')
          }
        }
      })
    }
  }
}
</script>
<style>
.title {
  background-color: #f4f4f4;
  font-size: 18px;
  padding-left: 20px;
  color: #397cc0;
  font-weight: 600;
  width: 100%;
  /* margin: 5px; */
}
</style>
