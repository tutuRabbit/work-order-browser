<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container>
      <!-- <j-form-container :disabled="formDisabled"> -->
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row v-if="!disVis">
          <a-col :span="8">
            <a-form-item label="年度保养计划" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag
                type="list"
                dictCode="year_plan_yearth"
                v-model.trim="model.keepYearth"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row v-if="!disVis">
          <a-button class="new-btn" @click="selectSparePart" type="primary">
            <a-icon type="plus" />
            选择设备
          </a-button>
          <!-- <a-button type="primary" @click="LevelVisible = true">
            <a-icon type="plus" />
            批设保养级别
          </a-button>
          <a-button type="primary" @click="handleStandard">
            <a-icon type="plus" />
            批设保养标准
          </a-button> -->
          <!-- <a-button>
            <a-icon type="plus" />
            删除
          </a-button> -->
          <j-vxe-table
            ref="vTable"
            row-number
            row-selection
            keep-source
            :height="500"
            :loading="personnel.loading"
            :dataSource="personnel.dataSource"
            :columns="personnel.columns"
            style="margin-top: 8px;"
          >
            <template v-slot:action="props">
              <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
                <a>删除</a>
              </a-popconfirm>
            </template>
          </j-vxe-table>
        </a-row>
        <!-- 编辑、查看 -->
        <a-row v-if="disVis">
          <a-col :span="24">
            <div style="font-size:20px;text-align:center;color:#000;font-weight:bold">
              {{ model.keepYearth }}年度{{ model.keepYearth }}保养计划
            </div>
          </a-col>
          <a-col :span="24"><div class="title" style="margin-top:20px">设备信息</div></a-col>
          <a-col :span="6">
            <a-form-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <span style="font-weight: 600;color: #000;">{{ model.equipmentCode }}</span>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="设备名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <span style="font-weight: 600;color: #000;">{{ model.equipmentName }}</span>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <span style="font-weight: 600;color: #000;">{{ model.equipmentModel }}</span>
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="设备类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-category-select
                style="width: 100%;font-weight: 600;color: #000;"
                v-model="model.equipmentType"
                :disabled="true"
                pcode="B03"
              />
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="使用部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-tree-select
                v-model="model.sysOrgCode"
                :disabled="true"
                show-search
                tree-default-expand-all
                tree-node-filter-prop="title"
                style="width: 100%;font-weight: 600;color: #000;"
                :tree-data="treeData"
                placeholder="请选择使用部门"
              />
            </a-form-item>
          </a-col>
          <a-col :span="6">
            <a-form-item label="设备位置" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <span style="font-weight: 600;color: #000;">{{ model.equipmentAddress }}</span>
            </a-form-item>
          </a-col>
          <a-col :span="24"><div class="title" style="margin-top:20px">保养标准</div></a-col>
          <div style="height:100px"></div>
          <a-col :span="8">
            <a-form-item label="保养级别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag
                type="list"
                :disabled="formDisabled"
                dictCode="keep_level"
                v-model.trim="queryParam.keepLevel"
                @change="$forceUpdate()"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item label="保养标准" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag
                @change="$forceUpdate()"
                dictCode="keep_standard"
                :disabled="formDisabled"
                v-model.trim="queryParam.keepStandard"
              ></j-dict-select-tag>
            </a-form-item>
          </a-col>
          <div style="height:20px"></div>
          <a-col :span="24"><div class="title" style="margin-top:20px">保养时间</div></a-col>
          <!-- 编辑 -->
          <j-vxe-table
            row-number
            row-selection
            :height="100"
            :disabled="formDisabled"
            :dataSource="editData.dataSource"
            :columns="editData.columns"
            style="margin-top: 18%;"
          />
          <a-col :span="24"><div class="title" style="margin-top:50px">操作记录</div></a-col>
          <j-vxe-table
            row-number
            row-selection
            :height="300"
            :disabled="formDisabled"
            :scroll="{ x: true }"
            style="padding-top: 7%"
            :columns="operateTable.columns"
            :dataSource="operateTable.dataSource"
          >
            <!-- <template slot="status" slot-scope="text, record">
              <a-button v-if="record.status == 0" type="primary" ghost>正常</a-button>
              <a-button v-if="record.status == 1" type="danger" ghost>异常</a-button>
            </template>
            <span slot="action" slot-scope="text, record">
              <a @click="handleView(record)"><a-icon type="eye" /> 详细</a>
            </span> -->
            <template v-slot:action="props">
              <a @click="handleView(props)"><a-icon type="eye" /> 详细</a>
            </template>
          </j-vxe-table>
          <!-- <a-table
            ref="operateTable"
            size="middle"
            bordered
            rowKey="id"
            :scroll="{ x: true }"
            :columns="operateTable.columns"
            :dataSource="operateTable.dataSource"
            class="j-table-force-nowrap"
          >
            <span slot="action" slot-scope="text, record">
              <a @click="handleView(record)"><a-icon type="eye" /> 详细</a>
            </span>
          </a-table> -->
        </a-row>
      </a-form-model>
    </j-form-container>
    <!-- 选择设备 -->
    <j-modal
      title="选择设备"
      width="60%"
      :visible="equipmentVis"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchEquipment">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="equipmentParam.search"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类别">
                <j-category-select v-model="equipmentParam.equipmentType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="equipmentParam.sysOrgCode"
                  style="width: 100%"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
                  :tree-data="treeData"
                  placeholder="请选择使用部门"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchEquipment" icon="search">查询</a-button>
                <a-button type="primary" @click="searchEquipmentReset" icon="reload" style="margin-left: 8px"
                  >重置</a-button
                >
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        row-selection-type="radio"
        rowKey="id"
        :columns="equipmentTable.columns"
        :dataSource="equipmentTable.dataSource"
        :pagination="equipmentTable.ipagination"
        :loading="equipmentTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleEquipmentTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
    <!-- 批设保养级别 -->
    <j-modal title="批设保养级别" width="600px" :visible="LevelVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="保养级别">
                <!-- <a-select v-model.trim="LevelParam.planTime" style="width: 100%" placeholder="请选择">
                    <a-select-option value="0">保养中</a-select-option>
                    <a-select-option value="1">外委中</a-select-option>
                  </a-select> -->
                <j-dict-select-tag
                  type="list"
                  dictCode="keep_level"
                  v-model.trim="LevelParam.keepLevel"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="LevelLoading" type="primary" @click="LevelOk">确认</a-button>
      </template>
    </j-modal>
    <!-- 批设保养标准 -->
    <j-modal title="批设保养标准" width="600px" :visible="standardVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <a-form-item label="保养标准">
                <a-select v-model.trim="standardParam.planTime" style="width: 100%" placeholder="请选择">
                  <a-select-option value="0">保养中</a-select-option>
                  <a-select-option value="1">外委中</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button :loading="LevelLoading" type="primary" @click="LevelOk">确认</a-button>
      </template>
    </j-modal>
    <!-- 操作记录 -->
    <j-modal title="操作记录详细" width="70%" :visible="detailVisible" @cancel="handleCancel" cancelText="关闭">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row>
            <a-col :span="12">
              <a-form-item label="操作模块：">{{ detailData.title }}</a-form-item>
              <a-form-item label="登录信息："
                >{{ detailData.operName }} / {{ detailData.operIp }} / {{ detailData.operLocation }}</a-form-item
              >
            </a-col>
            <a-col :span="12">
              <a-form-item label="请求地址：">{{ detailData.operUrl }}</a-form-item>
              <a-form-item label="请求方式：">{{ detailData.requestMethod }}</a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="操作方法：">{{ detailData.method }}</a-form-item>
              <a-form-item label="操作类型：">{{ detailData.businessTypeStr }}</a-form-item>
            </a-col>
            <a-col :span="24">
              <span>请求参数：</span>
              <span>{{ detailData.operParam }}</span>
              <!-- <a-form-item label="请求参数：">{{ detailData.operParam }}</a-form-item> -->
            </a-col>
            <a-col :span="24">
              <span style="margin-top:20px;display: block">返回参数：</span>
              <span style="margin-bottom:20px;display: block">{{ detailData.jsonResult }}</span>
              <!-- <a-form-item label="返回参数：">{{ detailData.jsonResult }}</a-form-item> -->
            </a-col>
            <a-col :span="12">
              <a-form-item label="操作状态：">
                <span>{{ detailData.statusStr }}</span>
                <!-- <div v-if="detailData.status === 0">正常</div>
                  <div v-else-if="detailData.status === 1">失败</div> -->
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item label="操作时间：">{{ detailData.operTime }}</a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="异常信息：" v-if="detailData.status === 1">{{ detailData.errorMsg }}</a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
      </template>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { queryDepartTreeList } from '@/api/api'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { FormTypes } from '@/utils/JEditableTableUtil'

export default {
  name: 'TabKeepYearPlanForm',
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
      queryParam: {},
      LevelParam: {},
      standardParam: {},
      LevelVisible: false,
      standardVisible: false,
      // 详细
      detailVisible: false,
      detailData: {},
      equipmentVis: false,
      disVis: false,
      LevelLoading: false,
      equipmentParam: {},
      selectedRowKeys: [],
      selectionRows: [],
      treeData: [],
      dataSource: [],
      editData: {
        dataSource: [],
        columns: [
          {
            title: '1月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month1'
          },
          {
            title: '2月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month2'
          },
          {
            title: '3月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month3'
          },
          {
            title: '4月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month4'
          },
          {
            title: '5月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month5'
          },
          {
            title: '6月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month6'
          },
          {
            title: '7月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month7'
          },
          {
            title: '8月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month8'
          },
          {
            title: '9月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month9'
          },
          {
            title: '10月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month10'
          },
          {
            title: '11月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month11'
          },
          {
            title: '12月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month12'
          }
        ]
      },
      operateTable: {
        dataSource: [],
        columns: [
          {
            title: '日志编号',
            align: 'center',
            type: FormTypes.normal,
            key: 'operId'
          },
          {
            title: '系统模块',
            align: 'center',
            type: FormTypes.normal,
            key: 'title'
          },
          {
            title: '操作类型',
            align: 'center',
            type: FormTypes.normal,
            key: 'businessTypeStr'
            // key: 'businessType'
            // 操作类别（0其它 1后台用户 2手机端用户）
          },
          {
            title: '请求方式',
            align: 'center',
            type: FormTypes.normal,
            key: 'requestMethod'
          },
          {
            title: '操作人员',
            align: 'center',
            type: FormTypes.normal,
            key: 'operName'
          },
          {
            title: '操作地址',
            align: 'center',
            type: FormTypes.normal,
            key: 'operIp'
          },
          {
            title: '操作状态',
            align: 'center',
            type: FormTypes.normal,
            key: 'statusStr'
            // customRender: function(t, r, index) {
            //   if (r.status == '0') {
            //     return (
            //       <a-button type="primary" ghost>
            //         正常
            //       </a-button>
            //     )
            //   } else {
            //     return (
            //       <a-button type="danger" ghost>
            //         异常
            //       </a-button>
            //     )
            //   }
            // }
          },
          {
            title: '操作日期',
            align: 'center',
            key: 'operTime',
            type: FormTypes.normal
          },
          {
            title: '操作',
            key: 'action',
            type: JVXETypes.slot,
            slotName: 'action',
            width: '147px',
            align: 'center',
            fixed: 'right'
          }
        ]
      },
      // operateTable: {
      //   dataSource: [],
      //   columns: [
      //     {
      //       title: '日志编号',
      //       align: 'center',
      //       dataIndex: 'operId'
      //     },
      //     {
      //       title: '系统模块',
      //       align: 'center',
      //       dataIndex: 'title'
      //     },
      //     {
      //       title: '操作类型',
      //       align: 'center',
      //       dataIndex: 'businessType',
      //       // 操作类别（0其它 1后台用户 2手机端用户）
      //     },
      //     {
      //       title: '请求方式',
      //       align: 'center',
      //       dataIndex: 'requestMethod'
      //     },
      //     {
      //       title: '操作人员',
      //       align: 'center',
      //       dataIndex: 'operName'
      //     },
      //     {
      //       title: '操作地址',
      //       align: 'center',
      //       dataIndex: 'operIp'
      //     },
      //     {
      //       title: '操作状态',
      //       align: 'center',
      //       dataIndex: 'status',
      //       customRender: function(t, r, index) {
      //         if (r.status == '0') {
      //           return (
      //             <a-button type="primary" ghost>
      //               正常
      //             </a-button>
      //           )
      //         } else {
      //           return (
      //             <a-button type="danger" ghost>
      //               异常
      //             </a-button>
      //           )
      //         }
      //       }
      //       // if (r.status == '0') {
      //       //   return '正常'
      //       // } else {
      //       //   return '异常'
      //       // }
      // 操作状态（0正常 1异常）
      //     },
      //     {
      //       title: '操作日期',
      //       align: 'center',
      //       dataIndex: 'operTime'
      //     },
      //     {
      //       title: '操作',
      //       dataIndex: 'action',
      //       align: 'center',
      //       fixed: 'right',
      //       width: 147,
      //       scopedSlots: { customRender: 'action' }
      //     }
      //   ]
      // },
      equipmentTable: {
        loading: false,
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
        columns: [
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
            title: '使用部门',
            align: 'center',
            dataIndex: 'depart'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentTypeName'
            // customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      columns: [
        {
          title: '1月',
          align: 'center',
          dataIndex: 'month1',
          scopedSlots: { customRender: 'iconSlot1' }
        },
        {
          title: '2月',
          align: 'center',
          dataIndex: 'month2',
          scopedSlots: { customRender: 'iconSlot2' }
        },
        {
          title: '3月',
          align: 'center',
          dataIndex: 'month3',
          scopedSlots: { customRender: 'iconSlot3' }
        },
        {
          title: '4月',
          align: 'center',
          dataIndex: 'month4',
          scopedSlots: { customRender: 'iconSlot4' }
        },
        {
          title: '5月',
          align: 'center',
          dataIndex: 'month5',
          scopedSlots: { customRender: 'iconSlot5' }
        },
        {
          title: '6月',
          align: 'center',
          dataIndex: 'month6',
          scopedSlots: { customRender: 'iconSlot6' }
        },
        {
          title: '7月',
          align: 'center',
          dataIndex: 'month7',
          scopedSlots: { customRender: 'iconSlot7' }
        },
        {
          title: '8月',
          align: 'center',
          dataIndex: 'month8',
          scopedSlots: { customRender: 'iconSlot8' }
        },
        {
          title: '9月',
          align: 'center',
          dataIndex: 'month9',
          scopedSlots: { customRender: 'iconSlot9' }
        },
        {
          title: '10月',
          align: 'center',
          dataIndex: 'month10',
          scopedSlots: { customRender: 'iconSlot10' }
        },
        {
          title: '11月',
          align: 'center',
          dataIndex: 'month11',
          scopedSlots: { customRender: 'iconSlot11' }
        },
        {
          title: '12月',
          align: 'center',
          dataIndex: 'month12',
          scopedSlots: { customRender: 'iconSlot12' }
        }
      ],
      personnel: {
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
            title: '设备id',
            width: '1%',
            key: 'deviceId',
            type: FormTypes.normal
          },
          {
            title: '设备编码',
            width: '8%',
            key: 'equipmentCode',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'equipmentName',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'equipmentModel',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '设备类型',
            key: 'equipmentTypeName',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '使用部门',
            key: 'depart',
            width: '8%',
            type: FormTypes.normal
          },
          {
            title: '保养级别',
            key: 'keepLevel',
            type: FormTypes.select,
            width: '8%',
            // options: [],
            dictCode: 'keep_level',
            placeholder: '请选择${title}'
          },
          {
            title: '保养标准',
            key: 'keepStandard',
            width: '8%',
            type: FormTypes.select,
            // options: [],
            dictCode: 'keep_standard',
            placeholder: '请选择${title}'
          },
          {
            title: '保养时间',
            width: '8%',
            key: 'keepTime',
            type: FormTypes.select,
            // options: [],
            // defaultValue: '3',
            dictCode: 'keep_plan_times',
            placeholder: '请输入${title}',
            defaultValue: '3',
            validateRules: [
              {
                required: true, // 必填
                message: '请输入${title}' // 显示的文本
              }
            ]
          },

          {
            title: '1月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month1'
          },
          {
            title: '2月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month2'
          },
          {
            title: '3月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month3'
          },
          {
            title: '4月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month4'
          },
          {
            title: '5月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month5'
          },
          {
            title: '6月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month6'
          },
          {
            title: '7月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month7'
          },
          {
            title: '8月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month8'
          },
          {
            title: '9月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month9'
          },
          {
            title: '10月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month10'
          },
          {
            title: '11月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month11'
          },
          {
            title: '12月',
            width: '5%',
            type: FormTypes.checkbox,
            key: 'month12'
          }
        ]
      },
      dictOptions: {},
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
        keepYearth: [{ required: true, message: '请选择年度保养计划!' }]
      },
      url: {
        add: '/year/tabKeepYearPlan/add',
        edit: '/year/tabKeepYearPlan/edit',
        equipmentList: '/equipment/equipment/list', //设备管理分页
        queryById: '/year/tabKeepYearPlan/queryById?id='
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    // 初始化默认年度保养计划
    let date = new Date()
    let year = date.getFullYear() //获取完整的年份(4位)
    this.model.keepYearth = Object(year)
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    /** 详细按钮操作 */
    handleView(row) {
      console.log('row', row)
      this.detailVisible = true
      this.detailData = row.row
    },
    LevelOk() {},
    // 设备管理
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    handleOk() {
      this.personnel.dataSource = this.selectionRows
      // console.log('this.personnel.dataSource', this.personnel.dataSource)
      this.handleCancel()
    },
    handleCancel() {
      this.equipmentVis = false
      this.LevelVisible = false
      this.standardVisible = false
      // 详细
      this.detailVisible = false
      this.detailData = {}
      this.LevelParam = {}
      this.equipmentParam = {}
      this.standardParam = {}
    },
    handleStandard() {
      let b = this.$refs.vTable.getSelectionData()
      let a = this.$refs.vTable.getTableData()
      console.log('a', a)
      console.log('b', b)
      // this.standardVisible = true
    },
    searchEquipment() {
      this.equipmentLoad(1)
    },
    searchEquipmentReset() {
      this.equipmentParam = {}
      this.equipmentLoad(1)
    },
    selectSparePart() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    handleEquipmentTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.equipmentTable.ipagination = pagination
      this.equipmentLoad()
    },
    equipmentLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.equipmentTable.ipagination.current = 1
      }
      var params = Object.assign(this.equipmentParam)
      params.model = 0
      params.pageNo = this.equipmentTable.ipagination.current
      params.pageSize = this.equipmentTable.ipagination.pageSize
      this.equipmentTable.loading = true
      getAction(this.url.equipmentList, params, this.isorter)
        .then(res => {
          if (res.success) {
            this.equipmentTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.equipmentTable.ipagination.total = res.result.total
            } else {
              this.equipmentTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.equipmentTable.loading = false
        })
    },
    add() {
      this.model = Object.assign({}, this.modelDefault)
    },
    edit(record) {
      this.searchList(record)
      this.disVis = true
      this.visible = true
    },
    show(record) {
      this.searchList(record)
      this.disVis = true
      this.visible = true
    },
    searchList(val) {
      getAction(this.url.queryById + val.id).then(res => {
        if (res.success) {
          let equipment = res.result.equipment
          let yearPlan = res.result.yearPlan
          let logs = res.result.logs
          this.model = equipment
          let pm = {
            month1: yearPlan.month1,
            month2: yearPlan.month2,
            month3: yearPlan.month3,
            month4: yearPlan.month4,
            month5: yearPlan.month5,
            month6: yearPlan.month6,
            month7: yearPlan.month7,
            month8: yearPlan.month8,
            month9: yearPlan.month9,
            month10: yearPlan.month10,
            month11: yearPlan.month11,
            month12: yearPlan.month12,
            id: yearPlan.id
          }
          this.queryParam.keepLevel = yearPlan.keepLevel
          this.queryParam.keepStandard = yearPlan.keepStandard
          this.dataSource.push(pm)
          this.editData.dataSource.push(pm)
          this.operateTable.dataSource = logs
          console.log('this.operateTable.dataSource', this.operateTable.dataSource)
        } else {
          this.dataSource = null
        }
      })
    },
    submitForm() {
      const that = this
      // 触发表单验证
      // 编辑
      if (this.disVis) {
        that.confirmLoading = true
        this.editData.dataSource[0].keepLevel = this.queryParam.keepLevel
        this.editData.dataSource[0].keepStandard = this.queryParam.keepStandard
        // console.log('this.editData.dataSource', this.editData.dataSource)
        httpAction(that.url.edit, this.editData.dataSource[0], 'put')
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
        // 新增
      } else {
        if (that.model.keepYearth == '' || that.model.keepYearth == undefined) {
          that.$message.warning('请选择年度保养计划！')
        } else if (that.personnel.dataSource.length == 0) {
          that.$message.warning('请选择设备添加计划！！')
        } else {
          this.$refs.vTable.validateTable().then(errMap => {
            if (!errMap) {
              that.confirmLoading = true
              if (!that.model.id) {
                let pm = {
                  yearPlans: this.personnel.dataSource,
                  keepYearth: this.model.keepYearth
                }
                httpAction(that.url.add, pm, 'post')
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
  }
}
</script>
<style>
.ant-btn-primary {
  margin-left: 10px;
}
/* .ant-col-8 {
  z-index: 9999;
} */
</style>
