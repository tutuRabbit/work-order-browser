<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model">
      <!-- 产品信息 -->
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">产品信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="产品编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productCode">
            <a-input :disabled="true" v-model="model.productCode" placeholder="请输入产品编码"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="产品名称"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="productName"
            :rules="[{ required: true, message: '请输入产品名称!' }]"
          >
            <a-input :disabled="true" v-model="model.productName" placeholder="请输入产品名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="产品规格" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productNorms">
            <a-input :disabled="true" v-model="model.productNorms" placeholder="请输入产品规格"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="产品属性" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="productProperty">
            <a-input :disabled="true" v-model="model.productProperty" placeholder="请输入产品属性"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sysOrgCode">
            <a-select
              :disabled="true"
              defaultValue="alipay"
              @change="fieldsChange(model.stockPiece)"
              placeholder="请选择"
              v-model="model.stockPiece"
            >
              <a-select-option v-for="(item, index) in stockPieceList" :key="index" :value="item.name"
                >{{ item.name }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="工艺路线" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="craftRouteId">
            <a-select :disabled="true" defaultValue="alipay" placeholder="请选择" v-model="model.craftRouteId">
              <a-select-option v-for="(item, index) in LineList" :key="index" :value="item.id"
                >{{ item.name }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="库存数量" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stockNumber">
            <a-input :disabled="true" v-model="model.stockNumber" placeholder="请输入库存数量"></a-input>
          </a-form-model-item>
        </a-col>
        <!-- <a-col :span="8">
          <a-form-model-item label="成品率(%)" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="address">
            <a-input :disabled="true" v-model="model.ddd" placeholder="请输入成品率"></a-input>
          </a-form-model-item>
        </a-col> -->
        <a-col v-if="!editDisabled" :span="8" style="margin-left:7%">
          <a-button type="primary" @click="selectProduct"> <a-icon type="search" /> 选择产品 </a-button>
        </a-col>
      </a-row>
      <a-row>
        <a-col :span="24">
          <div class="title" style="margin-top:20px">BOM信息</div>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="BOM单组别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="bomGroup">
            <a-input :disabled="formDisabled" v-model="model.bomGroup" placeholder="请输入BOM单组别" />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="BOM编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="bomCode">
            <a-input :disabled="formDisabled" v-model="model.bomCode" placeholder="请输入BOM编号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="版本" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="version">
            <a-input :disabled="formDisabled" v-model="model.version" placeholder="请输入BOM版本"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-radio-group v-model="model.status" :disabled="formDisabled">
              <a-radio value="0">使用</a-radio>
              <a-radio value="1">未使用</a-radio>
            </a-radio-group>
          </a-form-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="图号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="pictureNum">
            <a-input :disabled="formDisabled" v-model="model.pictureNum" placeholder="请输入图号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-item label="是否特性配置来源物料" :labelCol="labelA" :wrapperCol="wrapperA" prop="isFromMatter">
            <a-radio-group v-model="model.isFromMatter" :disabled="formDisabled">
              <a-radio value="0">是</a-radio>
              <a-radio value="1">否</a-radio>
            </a-radio-group>
          </a-form-item>
        </a-col>
        <a-col :span="8">
          <a-form-item label="跳层" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="isJumper">
            <a-radio-group v-model="model.isJumper" :disabled="formDisabled">
              <a-radio value="0">是</a-radio>
              <a-radio value="1">否</a-radio>
            </a-radio-group>
          </a-form-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
            <a-input :disabled="formDisabled" v-model="model.remark" placeholder="请输入备注"></a-input>
          </a-form-model-item>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- 子表单区域 -->
    <a-button v-if="!editDisabled" class="new-btn" @click="selectMaterial" type="primary">
      <a-icon type="plus" />
      选择物料
    </a-button>
    <j-vxe-table
      ref="editableTable1"
      row-number
      keep-source
      :height="300"
      :loading="table.loading"
      :dataSource="table.dataSource"
      :columns="table.columns"
      :disabled="editDisabled"
      style="margin-top: 8px;"
    >
      <template v-slot:action="props">
        <a-popconfirm title="确定删除吗？" v-if="!formDisabled" @confirm="handleDL(props)">
          <a>删除</a>
        </a-popconfirm>
      </template></j-vxe-table
    >
    <!-- 选择物料 -->
    <j-modal
      title="选择物料"
      :width="width"
      :visible="materialVis"
      @ok="materialOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">物料信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <!-- <a-form layout="inline" @keyup.enter.native="materialQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="产品编码">
                <a-input placeholder="请输入" v-model="queryParam.lastProductCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="产品名称">
                <a-input placeholder="请输入" v-model="queryParam.lastProductName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="materialQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="materialReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form> -->

        <j-super-query
          :fieldList="superFieldList"
          ref="superQueryModal"
          @handleSuperQuery="handleSuperQuery"
        ></j-super-query>
      </div>
      <a-table
        ref="table"
        style="margin-top: 10px"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="materialTable.columns"
        :dataSource="materialTable.dataSource"
        :pagination="materialTable.ipagination"
        :loading="materialTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: materialOnChange }"
        class="j-table-force-nowrap"
        @change="materialHandleTableChange"
      >
      </a-table>
    </j-modal>
    <!-- 产品信息 -->
    <j-modal
      title="产品信息"
      width="80%"
      :visible="productVis"
      @ok="productHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:10px ">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <!-- <a-form layout="inline" @keyup.enter.native="productQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="产品编码">
                <a-input placeholder="请输入" v-model="productParam.code"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="产品名称">
                <a-input placeholder="请输入" v-model="productParam.productName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="productQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="productReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form> -->
        <j-super-query
          :fieldList="productFieldList"
          ref="productQueryModal"
          @handleSuperQuery="handleProductQuery"
        ></j-super-query>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        style="margin-top:10px"
        :columns="productTable.columns"
        :dataSource="productTable.dataSource"
        :pagination="productTable.ipagination"
        :loading="productTable.loading"
        :rowSelection="{ selectedRowKeys: productRowKeys, onChange: productOnChange, type: 'radio' }"
        class="j-table-force-nowrap"
        @change="productHandleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { getRefPromise, validateFormModelAndTables } from '@/components/jeecg/JVxeTable/utils/vxeUtils'
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { filterObj } from '@/utils/util'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { mixinDevice } from '@/utils/mixin'
export default {
  name: 'TabBomForm',
  components: {},
  mixins: [JeecgListMixin, mixinDevice],
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
      // 选择产品
      productFieldList: [],
      productQueryParams: '',
      productQueryFlag: false,
      /** 高级查询拼接方式 */
      productQueryMatchType: 'and',
      // end
      superFieldList: [],
      productParam: {},
      productVis: false,
      productTable: {
        columns: [
          {
            title: '产品编码',
            align: 'center',
            dataIndex: 'code'
          },
          {
            title: '产品名称',
            align: 'center',
            dataIndex: 'productName'
          },
          {
            title: '产品规格',
            align: 'center',
            dataIndex: 'productNorms'
          },
          {
            title: '库存单位',
            align: 'center',
            dataIndex: 'stockPiece'
          },
          {
            title: '产品属性',
            align: 'center',
            dataIndex: 'productProperty'
          },
          {
            title: '工艺路线',
            align: 'center',
            dataIndex: 'craftRouteName'
          },
          {
            title: '最大库存',
            align: 'center',
            dataIndex: 'maxStock'
          },
          {
            title: '最小库存',
            align: 'center',
            dataIndex: 'minStock'
          },
          {
            title: '安全库存',
            align: 'center',
            dataIndex: 'securityStock'
          },
          {
            title: '库存数量',
            align: 'center',
            dataIndex: 'stockNumber'
          }
        ],
        dataSource: [],
        ipagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        loading: false
      },
      productRowKeys: [],
      productRows: [],
      // end
      // 选择物料
      materialVis: false,
      materialTable: {
        columns: [
          {
            title: '产品编码',
            align: 'center',
            dataIndex: 'lastProductCode'
          },
          {
            title: '产品名称',
            align: 'center',
            dataIndex: 'lastProductName'
          },
          {
            title: '产品规格',
            align: 'center',
            dataIndex: 'lastProductSpec'
          },
          {
            title: '单位',
            align: 'center',
            dataIndex: 'lastProductUnitName'
          },

          {
            title: '单位用量',
            align: 'center',
            dataIndex: 'unitQty'
          },
          {
            title: '备注',
            align: 'center',
            maxWidth: 500,
            ellipsis: true,
            dataIndex: 'remark'
          }
        ],
        dataSource: [],
        ipagination: {
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        loading: false
      },
      materialRowKeys: [],
      materialRows: [],
      // end
      equipmentParam: {},
      equipmentVis: false,
      stockPieceList: null,
      LineList: null,
      editDisabled: false,
      // 选择物料
      queryParam: {},
      width: 900,
      materialVis: false,
      allowClear: true,
      /* 分页参数 */
      ipagination: {
        current: 1,
        pageSize: 5,
        pageSizeOptions: ['5', '10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + '-' + range[1] + ' 共' + total + '条'
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      spareColumns: [
        {
          title: '备件编号',
          align: 'center',
          dataIndex: 'spareCode'
        },
        {
          title: '备件名称',
          align: 'center',
          dataIndex: 'spareName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'spareModel'
        },
        {
          title: '类型',
          align: 'center',
          dataIndex: 'spareType'
        },
        {
          title: '库存数',
          align: 'center',
          dataIndex: 'totalStockCount'
        },
        {
          title: '分库存',
          align: 'center',
          dataIndex: 'partStockCount'
        }
      ],
      disableSubmit: false,
      rowAmount: '',
      selectedRowKeys: [],
      selectionRows: [],
      //end
      title: '操作',
      visible: false,
      confirmLoading: false,
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 24 - 6 }
      },
      labelA: {
        xs: { span: 24 },
        sm: { span: 10 }
      },
      wrapperA: {
        xs: { span: 24 },
        sm: { span: 12 }
      },
      // 选择物料
      loading: false,
      dataSource: [],
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '80px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '物料编码',
            align: 'center',
            key: 'lastProductCode',
            type: FormTypes.normal
          },
          {
            title: '物料名称',
            align: 'center',
            key: 'lastProductName',
            type: FormTypes.normal
          },
          {
            title: '物料属性',
            align: 'center',
            type: FormTypes.normal
            // key: 'spareModel'
          },
          {
            title: '单位',
            align: 'center',
            key: 'lastProductUnitName',
            type: FormTypes.normal
          },
          {
            title: '基本单位',
            align: 'center',
            type: FormTypes.normal
            // key: 'count'
          },
          {
            title: '子产品名称',
            align: 'center',
            key: 'nextProductName',
            type: FormTypes.normal
          },
          {
            title: '子产品规格',
            align: 'center',
            key: 'nextProductSpec',
            type: FormTypes.normal
          },
          {
            title: '子单位',
            align: 'center',
            key: 'nextProductUnitName',
            type: FormTypes.normal
          },
          // {
          //   title: '子项类型',
          //   align: 'center',
          // type: FormTypes.normal
          //   // key: 'converUnit'
          // },
          {
            title: '基本单位用量',
            align: 'center',
            type: FormTypes.normal
            // key: 'converCount'
          },
          {
            title: '用量',
            align: 'center',
            type: FormTypes.normal
            // key: 'unitPrice'
          },
          {
            title: '损耗率(%)',
            align: 'center',
            type: FormTypes.normal
            // key: 'amount'
          },
          {
            title: '位置号',
            align: 'center',
            type: FormTypes.normal
            // key: 'amount'
          },
          {
            title: '胚料尺寸',
            align: 'center',
            type: FormTypes.normal
            // key: 'amount'
          },
          {
            title: '胚料数',
            align: 'center',
            type: FormTypes.normal
            // key: 'amount'
          },
          {
            title: '工序号',
            align: 'center',
            type: FormTypes.normal
            // key: 'amount'
          }
        ]
      },
      url: {
        list: '/longxianda/sysMaterial/list',
        add: '/bom/tabBom/add',
        edit: '/bom/tabBom/edit',
        queryById: '/bom/tabBom/queryById',
        productList: '/longxianda/sysProduct/list',
        materialList: '/longxianda/sysMaterial/list'
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    this.loadStockPieceList()
    this.getProductFieldList()
    this.getSuperFieldList()
    this.loadLineList()
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    // getAction('/detail/modelDetail/queryStockDetails').then(res => {
    //   console.log('getAction', res)
    //   if (res.success) {
    //     this.dataSource = res.result.records || []
    //   }
    // })
  },
  methods: {
    getQueryField() {
      //TODO 字段权限控制
      var str = 'id,'
      this.materialTable.columns.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    getProductQueryField() {
      //TODO 字段权限控制
      var str = 'id,'
      this.productTable.columns.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    handleSuperQuery(params, matchType) {
      //高级查询方法
      if (!params) {
        this.superQueryParams = ''
        this.superQueryFlag = false
      } else {
        this.superQueryFlag = true
        this.superQueryParams = JSON.stringify(params)
        this.superQueryMatchType = matchType
      }
      // this.loadData(1)
      console.log(' this.queryParam', this.queryParam)
      console.log('matchType', matchType)
      console.log('params', params)
      this.materialLoad(1)
    },
    handleProductQuery(params, matchType) {
      //高级查询方法
      if (!params) {
        this.productQueryParams = ''
        this.productQueryFlag = false
      } else {
        this.productQueryFlag = true
        this.productQueryParams = JSON.stringify(params)
        this.productQueryMatchType = matchType
      }
      this.productLoad(1)
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      param.field = this.getQueryField()
      param.pageNo = this.materialTable.ipagination.current
      param.pageSize = this.materialTable.ipagination.pageSize
      return filterObj(param)
    },
    getProductParams() {
      //获取查询条件
      let sqp = {}
      if (this.productQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.productQueryParams)
        sqp['superQueryMatchType'] = this.productQueryMatchType
      }
      var param = Object.assign(sqp, this.productParam, this.isorter, this.filters)
      param.field = this.getProductQueryField()
      param.pageNo = this.productTable.ipagination.current
      param.pageSize = this.productTable.ipagination.pageSize
      return filterObj(param)
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'lastProductCode', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductSpec', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'lastProductUnitName', text: '单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'unitQty', text: '单位用量', dictCode: '' })
      fieldList.push({ type: 'string', value: 'remark', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    },
    getProductFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'code', text: '产品编码', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productName', text: '产品名称', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productNorms', text: '产品规格', dictCode: '' })
      fieldList.push({ type: 'string', value: 'stockPiece', text: '库存单位', dictCode: '' })
      fieldList.push({ type: 'string', value: 'productProperty', text: '产品属性', dictCode: '' })
      fieldList.push({ type: 'string', value: 'craftRouteName', text: '工艺路线', dictCode: '' })
      this.productFieldList = fieldList
    },
    handleDL(props) {
      // 调用删除方法
      console.log('props.row', props.row)
      props.target.removeRows(props.row)
      let id = props.row.id
      for (var i = 0; i < this.table.dataSource.length; i++) {
        if (this.table.dataSource[i].id == id) {
          this.table.dataSource.splice(i, 1)
        }
      }

      // this.table.dataSource.forEach(item => {
      //   if (item.id == id) {
      //     delete item
      //   } else {
      //   }
      // })
      // console.log(' this.table.dataSource', this.table.dataSource)
    },
    // 产品信息
    productOnChange(productRowKeys, productRows) {
      this.productRowKeys = productRowKeys
      this.productRows = productRows
    },
    productHandleOk() {
      if (this.productRows.length != 0) {
        this.model.productId = this.productRows[0].id
        this.model.productCode = this.productRows[0].code
        this.model.productName = this.productRows[0].productName
        this.model.productNorms = this.productRows[0].productNorms
        this.model.productProperty = this.productRows[0].productProperty
        this.model.stockPiece = this.productRows[0].stockPiece
        this.model.craftRouteId = this.productRows[0].craftRouteId
        this.model.stockNumber = this.productRows[0].stockNumber
      }
      this.productVis = false
    },
    // end
    // 获取单位列表
    loadStockPieceList() {
      var that = this
      let httpurl = '/longxianda/sysSpec/list'
      let pm = { pageNo: 1, pageSize: 10000 }
      getAction(httpurl, pm)
        .then(res => {
          if (res.success) {
            that.stockPieceList = res.result.records
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
    // 选择产品
    productQuery() {
      this.productLoad(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    productReset() {
      this.productParam = {}
      this.productLoad(1)
    },
    selectProduct() {
      this.productLoad()
      this.productVis = true
    },
    productLoad(arg) {
      if (!this.url.productList) {
        this.$message.error('请设置url.productList属性!')
        return
      }
      if (arg === 1) {
        this.productTable.ipagination.current = 1
      }
      var params = this.getProductParams() //查询条件
      // var params = Object.assign(this.productParam)
      // params.pageNo = this.productTable.ipagination.current
      // params.pageSize = this.productTable.ipagination.pageSize
      this.productTable.loading = true
      getAction(this.url.productList, params)
        .then(res => {
          if (res.success) {
            this.productTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.productTable.ipagination.total = res.result.total
            } else {
              this.productTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.productTable.loading = false
        })
    },
    // 选择产品end
    // 选择物料
    materialQuery() {
      this.materialLoad(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    materialReset() {
      this.queryParam = {}
      this.materialLoad(1)
    },
    materialHandleTableChange(pagination) {
      //分页、排序、筛选变化时触发
      this.materialTable.ipagination = pagination
      this.materialLoad()
    },
    materialOnChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    selectMaterial() {
      this.materialLoad()
      this.materialVis = true
    },
    materialLoad(arg) {
      if (!this.url.materialList) {
        this.$message.error('请设置url.materialList属性!')
        return
      }
      if (arg === 1) {
        this.materialTable.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      // var params = {
      //   lastProductCode: this.queryParam.lastProductCode,
      //   lastProductName: this.queryParam.lastProductName,
      //   pageNo: this.materialTable.ipagination.current,
      //   pageSize: this.materialTable.ipagination.pageSize
      // }
      this.materialTable.loading = true
      getAction(this.url.materialList, params)
        .then(res => {
          if (res.success) {
            this.materialTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.materialTable.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.materialTable.loading = false
        })
    },
    // 选择物料end
    // end
    productHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      this.productTable.ipagination = pagination
      this.productLoad()
    },
    // 获取所有的editableTable实例
    getAllTable() {
      return Promise.all([getRefPromise(this, 'editableTable1'), getRefPromise(this, 'editableTable2')])
    },

    add() {
      // this.edit(this.modelDefault)
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      getAction('/bom/tabBom/queryById?id=' + record.id).then(res => {
        if (res.success) {
          // bom:bom信息;product产品信息;materials物料信息
          this.model = Object.assign({}, res.result.bom)
          this.table.dataSource = res.result.materials || []
          if (res.result.product) {
            let product = res.result.product
            this.model.productCode = product.productCode
            this.model.productName = product.productName
            this.model.productNorms = product.productNorms
            this.model.productProperty = product.productProperty
            this.model.stockPiece = product.stockPiece
            this.model.stockNumber = product.stockNumber
            this.model.craftRouteId = product.craftRouteId
            this.model.productCode = product.code
          }
          console.log(' this.table.dataSource', this.table.dataSource)
          console.log('res.result.materials ', res.result.materials)
        }
      })
      this.visible = true
      this.editDisabled = true
    },
    submitForm() {
      const that = this
      console.log(' this.model.productName', this.model.productName)
      console.log(' this.table.dataSource', this.table.dataSource)
      // 触发表单验证
      // this.$refs.form.validate(valid => {
      if (this.model.productName) {
        // if (valid) {
        that.confirmLoading = true
        let httpurl = ''
        let method = ''
        console.log('this.model.id', this.model.id)
        if (!this.model.id) {
          httpurl += this.url.add
          method = 'post'
        } else {
          httpurl += this.url.edit
          method = 'put'
        }
        that.model.materialIds = ''
        if (that.table.dataSource.length != 0) {
          that.model.materialIds = ''
          that.table.dataSource.forEach(val => {
            that.model.materialIds += val.id + ','
          })
          that.model.materialIds =
            that.model.materialIds.substring(that.model.materialIds.length - 1) == ','
              ? that.model.materialIds.substring(0, that.model.materialIds.length - 1)
              : that.model.materialIds
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
        this.$message.warning('请选择产品!')
      }
      // })
    },
    close() {
      this.visible = false
      this.materialVis = false
      this.productVis = false
      this.productParam = {}
      this.equipmentVis = false
      this.getAllTable().then(editableTables => {
        this.table.dataSource = []
      })
      this.$emit('close')
    },
    /** 查询某个tab的数据 */
    requestTableData(url, params, tab) {
      tab.loading = true
      getAction(url, params)
        .then(res => {
          tab.dataSource = res.result || []
        })
        .finally(() => {
          tab.loading = false
        })
    },
    materialOk() {
      this.selectionRows.forEach(item => {
        item.unitPrice = item.sparePrice
      })
      this.table.dataSource = this.selectionRows
      this.materialVis = false
      console.log(' this.table.dataSource', this.table.dataSource)
      console.log('this.selectionRows', this.selectionRows)
    },
    handleCancel() {
      this.close()
    },

    /** 触发表单验证 */
    validateFields() {
      this.getAllTable()
        .then(tables => {
          /** 一次性验证主表和所有的次表 */
          return validateFormModelAndTables(this.$refs.form, this.model, tables)
        })
        .then(allValues => {
          let formData = this.classifyIntoFormData(allValues)
          // 发起请求
          return this.requestAddOrEdit(formData)
        })
        .catch(e => {
          console.error(e)
        })
    },
    /** 整理成formData */
    classifyIntoFormData(allValues) {
      let orderMain = Object.assign(this.model, allValues.formValue)
      return {
        ...orderMain, // 展开
        jeecgOrderCustomerList: allValues.tablesValue[0].tableData,
        jeecgOrderTicketList: allValues.tablesValue[1].tableData
      }
    },
    /** 发起新增或修改的请求 */
    requestAddOrEdit(formData) {
      let url = this.url.add,
        method = 'post'
      if (this.model.id) {
        url = this.url.edit
        method = 'put'
      }
      this.confirmLoading = true
      httpAction(url, formData, method)
        .then(res => {
          if (res.success) {
            this.$message.success(res.message)
            this.$emit('ok')
            this.close()
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.confirmLoading = false
        })
    }
  }
}
</script>

<style lang="less" scoped>
/** Button按钮间距 */
// .ant-btn {
//   margin-left: 30px;
//   margin-bottom: 30px;
//   float: right;
// }
.drawer-footer {
  position: absolute;
  bottom: -8px;
  width: 100%;
  border-top: 1px solid #e8e8e8;
  padding: 10px 16px;
  text-align: right;
  left: 0;
  background: #fff;
  border-radius: 0 0 2px 2px;
}
</style>
