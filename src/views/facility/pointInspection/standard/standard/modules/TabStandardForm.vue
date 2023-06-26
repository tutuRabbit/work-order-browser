<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <!--  设备信息-->
      <a-row>
        <a-col :span="24">
          <div class="title">巡检标准</div>
        </a-col>
        <a-col :span="12">
          <a-form-model-item
            label="标准名称"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 4 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 20 }
            }"
            prop="standardName"
          >
            <a-input :disabled="formDisabled" v-model="model.standardName" placeholder="请输入标准名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="设备类型"
            :labelCol="{
              xs: { span: 24 },
              sm: { span: 4 }
            }"
            :wrapperCol="{
              xs: { span: 24 },
              sm: { span: 8 }
            }"
            prop="equipmentType"
          >
            <j-category-select
              :disabled="formDisabled"
              v-model="model.equipmentType"
              @select="handleSelect"
              pcode="B03"
              placeholder="请选择设备类型"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="工具" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="tool">
            <a-textarea :disabled="formDisabled" v-model="model.tool" placeholder="请输入工具"></a-textarea>
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="巡检目的" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="patrolInspection">
            <a-textarea
              :disabled="formDisabled"
              v-model="model.patrolInspection"
              placeholder="请输入巡检目的"
            ></a-textarea>
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-textarea :disabled="formDisabled" v-model="model.remark" placeholder="请输入备注"></a-textarea>
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="管理图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-upload
              :disabled="formDisabled"
              list-type="picture-card"
              :file-list="manageImage"
              :beforeUpload="beforeUploadMains"
              @change="handleChangeImg"
              :showUploadList="{
                showPreviewIcon: false
              }"
            >
              <div v-if="this.manageImage.length < 5">
                <!-- 确定上传图片数量 -->
                <a-icon type="plus" />
                <div class="ant-upload-text">上传</div>
              </div>
            </a-upload>
          </a-form-model-item>
        </a-col>
        <a-col :span="24">
          <a-form-model-item label="标准" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-button v-if="!formDisabled" class="new-btn" @click="selectSparePart" type="primary">
              <a-icon type="plus" />
              添加标准
            </a-button>
            <j-vxe-table
              ref="standard"
              row-number
              :rowSelection="false"
              keep-source
              :height="260"
              :loading="standard.loading"
              :dataSource="standard.dataSource"
              :columns="standard.columns"
              :disabled="true"
              style="margin-top: 8px;"
            >
              <template v-slot:action="props">
                <a v-if="!formDisabled" @click="handleEdit(props)">修改</a>
                <a-divider v-if="!formDisabled" type="vertical" />
                <a-popconfirm v-if="!formDisabled" title="确定删除吗?" @confirm="handleDL(props)">
                  <a>删除</a>
                </a-popconfirm>
              </template>
              <template v-slot:recordingMethod="props">
                <span>{{
                  props.row.recordingMethod == '0'
                    ? '单选'
                    : props.row.recordingMethod == '1'
                    ? '多选'
                    : props.row.recordingMethod == '2'
                    ? '数字'
                    : '文本'
                }}</span>
              </template>
            </j-vxe-table>
          </a-form-model-item>
        </a-col>
      </a-row>
      <a-col :span="24"> </a-col>
    </a-form-model>
    <!-- 添加标准 -->
    <j-modal
      title="添加标准"
      width="50%"
      :visible="standardVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form-model ref="standardForm" :model="searchData" :rules="searchRules">
          <a-row>
            <a-col :span="24">
              <div class="title">巡检标准</div>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                label="检查项"
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 2 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                prop="name"
              >
                <a-input v-model="searchData.name" placeholder="请输入检查项"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                label="记录方式"
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 2 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
              >
                <a-select style="width:30%" v-model="searchData.recordingMethod" @change="recordingChange">
                  <a-select-option value="0">单选</a-select-option>
                  <a-select-option value="1">多选</a-select-option>
                  <a-select-option value="2">数字</a-select-option>
                  <a-select-option value="3">文本</a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="24" v-if="searchData.recordingMethod == '0' || searchData.recordingMethod == '1'">
              <a-form-model-item
                label="正常值"
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 2 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                prop="normalValue"
              >
                <a-textarea v-model="searchData.normalValue" placeholder="请输入工具"></a-textarea>
              </a-form-model-item>
            </a-col>
            <a-col :span="24" v-if="searchData.recordingMethod == '0' || searchData.recordingMethod == '1'">
              <a-form-model-item
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 2 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                label="异常值"
                prop="qutlier"
              >
                <a-textarea v-model="searchData.qutlier" placeholder="请输入巡检目的"></a-textarea>
              </a-form-model-item>
            </a-col>
            <a-col :span="12" v-if="searchData.recordingMethod == '2'">
              <a-form-model-item
                label="下限"
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 4 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                prop="lowerLimit"
              >
                <a-input-number
                  @change="$forceUpdate()"
                  style="width:80%"
                  v-model="searchData.lowerLimit"
                  placeholder="请输入下限"
                ></a-input-number>
              </a-form-model-item>
            </a-col>
            <a-col :span="12" v-if="searchData.recordingMethod == '2'">
              <a-form-model-item
                label="上限"
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 4 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                prop="upperLimit"
              >
                <a-input-number
                  @change="$forceUpdate()"
                  style="width:80%"
                  v-model="searchData.upperLimit"
                  placeholder="请输入上限"
                ></a-input-number>
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                :labelCol="{
                  xs: { span: 24 },
                  sm: { span: 2 }
                }"
                :wrapperCol="{
                  xs: { span: 24 },
                  sm: { span: 20 }
                }"
                label="默认值"
                prop="acquiesce"
              >
                <a-input style="width:100%" v-model="searchData.acquiesce" placeholder="请输入备注"></a-input>
              </a-form-model-item>
            </a-col>
          </a-row>
        </a-form-model>
      </div>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'
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
      standardParam: {},
      standardVis: false,
      inspectionstandardList: [],
      fileList: [],
      manageImage: [],
      linkedId: '',
      disEdit: false,
      model: {
        // timeType: 'time1'
      },
      queryParam: {
        docType: '2'
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 2 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 20 }
      },
      confirmLoading: false,
      validatorRules: {
        standardName: [{ required: true, message: '请输入标准名称' }]
      },
      searchRules: {
        name: [{ required: true, message: '请输入检查项' }]
      },
      previewImage: '',
      previewVisible: false,
      url: {
        add: '/checkout/pointpatrolstandard/add',
        edit: '/checkout/pointpatrolstandard/edit',
        // list: '/spare/tabSpare/queryById',
        queryById: '/checkout/pointpatrolstandard/findById?id=',
        addPartPicBind: '/attachment/tabAttachment/addPartPicBind', //添加图片
        addPartDocBind: '/attachment/tabAttachment/addPartDocBind' //添加文档
      },
      standard: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '操作',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '100px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '检查项',
            key: 'name',
            type: FormTypes.normal
          },
          {
            title: '结果类型',
            key: 'recordingMethod',
            type: JVXETypes.slot,
            slotName: 'recordingMethod'
          },
          {
            title: '结果选项',
            key: 'standardModel',
            width: '20%',
            type: FormTypes.normal
          },
          {
            title: '下限',
            key: 'lowerLimit',
            type: FormTypes.normal
          },
          {
            title: '上限',
            key: 'upperLimit',
            type: FormTypes.normal
          },
          {
            title: '默认值',
            key: 'acquiesce',
            type: FormTypes.normal
          }
        ]
      },
      index: '',

      oldData: {},
      searchData: {}
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
  beforeCreate() {
    this.disableMixinCreated = true
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
  },
  methods: {
    handleEdit(val) {
      this.index = val.index
      this.searchData = val.row
      // this.oldData = this.searchData
      this.standardVis = true
    },
    handleDL(val) {
      this.standard.dataSource.splice(val.index, 1)
    },
    recordingChange() {
      this.searchData.normalValue = ''
      this.searchData.qutlier = ''
      this.searchData.lowerLimit = ''
      this.searchData.upperLimit = ''
      // this.searchData.acquiesce = ''
      this.$forceUpdate()
    },
    handleSelect(value, node, extray) {
      if (node.isLeaf == false) {
        this.$message.warning('请选择子项！')
      }
      this.model.equipmentType = node.eventKey
      this.model.equipmentTypeName = node.title
      this.$forceUpdate()
    },
    handleOk() {
      var that = this
      if (that.searchData.recordingMethod == '0' || that.searchData.recordingMethod == '1') {
        that.searchData.normalValue ? that.searchData.normalValue : '正常'
        that.searchData.qutlier ? that.searchData.qutlier : '异常'
        let a = that.searchData.normalValue.replace('\n', '/')
        let b = that.searchData.qutlier.replace('\n', '/')
        that.searchData.standardModel = a + ':' + b
      }
      if (that.standard.dataSource.length != 0) {
        if (!that.index) {
          that.standard.dataSource.push(that.searchData)
        } else {
          that.standard.dataSource[that.index] = that.searchData
        }
      } else {
        that.standard.dataSource.push(that.searchData)
      }
      this.standardVis = false
      this.searchData = {}
      this.index = ''
    },
    handleCancel() {
      this.standardVis = false
      this.searchData = {}
      // console.log('this.oldData', this.oldData)
      // console.log('this.searchData', this.searchData)
      // console.log('this.oldData', this.oldData.qutlier)
      // console.log('this.oldData', this.searchData.qutlier)
      this.index = ''
    },
    // 添加标准
    selectSparePart() {
      this.searchData.recordingMethod = '0'
      this.standardVis = true
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
      this.manageImage = fileList
    },
    // 预览
    handlePreview(file) {
      this.previewImage = file.url || file.thumbUrl
      this.previewVisible = true
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
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      getAction(this.url.queryById + record.id).then(res => {
        if (res.code == 200) {
          this.model = res.data
          this.standard.dataSource = res.data.inspectionstandardList ? res.data.inspectionstandardList : []
        }
      })
      this.disEdit = true
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          this.standard.dataSource.forEach(item => {
            delete item.id
          })
          this.model.inspectionstandardList = this.standard.dataSource
          httpAction(httpurl, this.model, method).then(res => {
            if (res.code == 200) {
              if (res.data) {
                this.queryParam.linkedId = res.result
              }
              that.$message.success(res.msg)
              this.standardVis = false
              this.searchData = {}
              this.index = ''
              that.$emit('ok')
            } else {
              that.$message.warning(res.msg)
            }
          })
          // setTimeout(() => {
          //   // 关联文档
          //   if (this.fileList.length != 0) {
          //     var pm = new FormData()
          //     if (this.fileList.length != 0) {
          //       let { fileList } = this
          //       fileList.forEach(file => {
          //         pm.append('files', file)
          //       })
          //     }
          //     pm.append('docType', this.queryParam.docType)
          //     pm.append('linkedId', this.queryParam.linkedId)
          //     postAction(this.url.addPartDocBind, pm)
          //       .then(res => {
          //         if (res.success) {
          //           that.$message.success(res.msg)
          //           setTimeout(() => {
          //             that.$emit('ok')
          //           }, 200)
          //         } else {
          //           that.$message.warning(res.msg)
          //         }
          //       })
          //       .finally(() => {
          //         that.confirmLoading = false
          //       })
          //   }
          //   //关联图片
          //   if (this.manageImage.length != 0) {
          //     var pm = new FormData()
          //     if (this.manageImage.length > 0) {
          //       this.manageImage.forEach(v => {
          //         pm.append('pictures', v.originFileObj)
          //       })
          //     }
          //     pm.append('linkedId', this.queryParam.linkedId)
          //     postAction(this.url.addPartPicBind, pm)
          //       .then(res => {
          //         if (res.success) {
          //           that.$message.success(res.msg)
          //           setTimeout(() => {
          //             that.$emit('ok')
          //           }, 200)
          //         } else {
          //           that.$message.warning(res.msg)
          //         }
          //       })
          //       .finally(() => {
          //         that.confirmLoading = false
          //       })
          //   }
          // }, 500)
          if (this.fileList.length == 0) {
            that.$emit('ok')
          }
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
.title {
  background-color: #f4f4f4;
  font-size: 18px;
  padding-left: 20px;
  color: #397cc0;
  font-weight: 600;
  width: 100%;
}
</style>
