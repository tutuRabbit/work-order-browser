<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <div class="title">设备信息</div>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentCode">
              <a-input v-model="model.equipmentCode" placeholder="不填可自动生成"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentName">
              <a-input v-model="model.equipmentName" placeholder="请输入设备名称"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentType">
              <j-category-select
                v-model="model.equipmentType"
                @select="handleSelect"
                pcode="B03"
                placeholder="请选择设备类型"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentModel">
              <a-input v-model="model.equipmentModel" placeholder="请输入规格型号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="生产厂商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentFirm">
              <a-select
                show-search
                :filterOption="filterOption"
                v-model="model.equipmentFirm"
                placeholder="回车可搜索"
                style="width: 100%"
              >
                <a-select-option v-for="(item, index) in equipmentSupplierOptions" :key="index" :value="item.id">
                  {{ item.companyName }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="供应商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentSupplier">
              <a-select
                show-search
                :filterOption="filterOption"
                v-model="model.equipmentSupplier"
                placeholder="回车可搜索"
                style="width: 100%"
              >
                <a-select-option v-for="(item, index) in equipmentSupplierOptions" :key="index" :value="item.id">
                  {{ item.companyName }}
                </a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="使用寿命" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentAge">
              <a-input-number v-model="model.equipmentAge" placeholder="请输入使用寿命" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="购买时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentBuyTime">
              <j-date
                placeholder="请选择购买时间"
                v-model="model.equipmentBuyTime"
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                style="width: 100%"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="资产原值" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="orignalCost">
              <a-input-number v-model="model.orignalCost" placeholder="请输入资产原值" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="资产净值" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="nav">
              <a-input-number v-model="model.nav" placeholder="请输入资产净值" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="资产负责人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentManager">
              <a-input v-model="model.equipmentManager" placeholder="请输入资产负责人"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备标记" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentMarks">
              <a-checkbox-group v-model="model.equipmentMarks">
                <a-row :gutter="24">
                  <a-col :span="6" v-for="{ value, label } in Options" :key="value">
                    <a-checkbox :value="value">
                      {{ label }}
                    </a-checkbox>
                  </a-col>
                </a-row>
              </a-checkbox-group>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <div class="title">使用状况</div>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="使用状况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentStatus">
              <j-dict-select-tag
                type="list"
                v-model="model.equipmentStatus"
                dictCode="equipment_use_status"
              ></j-dict-select-tag>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="使用部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sysOrgCode">
              <a-tree-select
                v-model="model.sysOrgCode"
                style="width: 100%"
                :tree-data="treeData"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                placeholder="请选择使用部门"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="操作人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentOperator">
              <!-- <a-input v-model="model.equipmentOperator" placeholder="请输入操作人"></a-input> -->
              <j-dict-select-tag
                v-model="model.equipmentOperator"
                placeholder="请选择操作人"
                dictCode="sys_user,realname,id"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="维修组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="depart">
              <!-- <j-dict-select-tag
                type="list"
                v-model="model.depart"
                dictCode="maintain_list"
                placeholder="请选择维修组"
              /> -->
              <a-tree-select
                v-model="model.depart"
                style="width: 100%"
                :tree-data="treeData"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                placeholder="请选择维修组"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="设备位置" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentAddress">
              <a-input v-model="model.equipmentAddress" placeholder="请输入设备位置"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="8">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remarks">
              <a-input type="textarea" v-model="model.remarks" placeholder="请输入备注"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { queryDepartTreeList } from '@/api/api'
import { validateDuplicateValue } from '@/utils/util'

export default {
  name: 'EquipmentForm',
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
      model: {},
      equipmentSupplierOptions: [],
      loadedKeys: [],
      treeDataType: [],
      replaceFields: {
        children: 'children',
        title: 'title',
        key: 'key'
      },
      treeData: [],
      Options: [
        // { label: '重点设备', value: 1 },
        // { label: '主要设备', value: 2 },
        // { label: '一般设备', value: 3 }
        { label: '重点设备', value: '重点设备' },
        { label: '主要设备', value: '主要设备' },
        { label: '一般设备', value: '一般设备' }
      ],
      check1: false,
      check2: false,
      check3: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {
        equipmentName: [{ required: true, message: '请输入设备名称!' }],
        equipmentType: [{ required: true, message: '请选择设备类型!' }],
        sysOrgCode: [{ required: true, message: '请选择使用部门!' }]
      },
      url: {
        add: '/equipment/equipment/add',
        list: '/sys/category/loadTreeData', //设备类型
        edit: '/equipment/equipment/edit',
        queryById: '/equipment/equipment/queryById'
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
    getAction('/sys/category/loadTreeData', { pcode: 'B03' }).then(res => {
      if (res.success) {
        let arr = [...res.result]
        console.log('这个方法是找到所有的部门信息', arr)
        arr = arr.map(item => {
          if (item.leaf == false) {
            item.isLeaf = false
          } else if (item.leaf == true) {
            item.isLeaf = true
          }
          return item
        })
        // arr.forEach(item => {
        //   item.isLeaf = false
        //   this.$message.warning('请选择子项！')
        // })
        this.treeDataType = arr
      }
    })
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        // console.log('arr', arr)
        // arr.forEach(item => {
        //   item.isLeaf = false
        //   this.$message.warning('请选择子项！')
        // })
        this.treeData = arr
      }
    })
    //生产商、供应商
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        this.equipmentSupplierOptions = arr
      }
    })
  },
  methods: {
    // 异步加载数据
    onLoadDeptData(treeNode) {
      console.log('treeNode.dataRef', treeNode.dataRef)
      return new Promise(resolve => {
        setTimeout(() => {
          let pm = {
            pid: treeNode.dataRef.key
          }
          getAction(this.url.list, pm).then(res => {
            console.log('res.result', res.result)
            treeNode.dataRef.children = res.result.map(item => {
              if (item.leaf == false) {
                item.isLeaf = false
              } else if (item.leaf == true) {
                item.isLeaf = true
              }
              return item
            })
            this.treeDataType = [...this.treeDataType]
            resolve()
          }, 1000)
        })
      })
    },
    loadDept(loadedKeys) {
      this.loadedKeys = loadedKeys
    },
    changeType(item) {
      console.log('item', item)
      console.log(' item.isLeaf', item.isLeaf)
    },
    handleSelect(value, node, extray) {
      if (node.isLeaf == false) {
        this.$message.warning('请选择子项！')
      }
      console.log('==>node', node) // 该选中节点的属性
      console.log('==>node', node.isLeaf) // 该选中节点的属性
      console.log('node.key', node.key)
      console.log('node.eventKey', node.eventKey)
      console.log('node.key', node.title)
      this.model.equipmentType = node.eventKey
      this.model.equipmentTypeName = node.title
      console.log('this.model', this.model)
      console.log('this.model.equipmentType', this.model.equipmentType)
      console.log('this.model.equipmentTypeName', this.model.equipmentTypeName)
      this.$forceUpdate()
    },
    filterOption(value, option) {
      return option.componentOptions.children[0].text.indexOf(value) >= 0
    },
    add() {
      // this.edit(this.modelDefault)
      this.visible = true
    },
    edit(record) {
      // this.model = Object.assign({}, record)
      getAction('/equipment/equipment/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.model = res.result
        }
        // this.model.equipmentMarks
        console.log('this.model.equipmentMarks', this.model.equipmentMarks)
        // debugger
      })
      //
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
<style lang="less" scoped>
.ant-modal-body {
  max-height: 700px;
  overflow-y: auto;
}
.title {
  background-color: #f4f4f4;
  font-size: 18px;
  padding-left: 20px;
  color: #397cc0;
  font-weight: 600;
  width: 100%;
  /* margin: 5px; */
}
.ant-col-6 {
  width: 50%;
}
</style>
