<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model" :rules="validatorRules">
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">基本信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item
            label="计划名称"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            :rules="[{ required: true, message: '请输入计划名称!' }]"
            prop="planName"
          >
            <a-input :disabled="formDisabled" v-model="model.planName" placeholder="请输入计划名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="计划所属部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planDepart">
            <a-tree-select
              v-model.trim="model.planDepart"
              style="width: 100%"
              :disabled="formDisabled"
              show-search
              tree-default-expand-all
              tree-node-filter-prop="title"
              :tree-data="treeData"
              placeholder="请选择"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="巡检周期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="checkCycle">
            <a-input-number
              :disabled="formDisabled"
              v-model.trim="model.checkCycle"
              style="width: 40%;"
              placeholder="请输入"
            >
            </a-input-number>
            <j-dict-select-tag
              style="width: 40%;margin-left:10px"
              type="list"
              dictCode="time_type"
              :disabled="formDisabled"
              v-model.trim="model.checkCycleTimeUnit"
            ></j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="计划巡检开始时间"
            :rules="[{ required: true, message: '请选择计划巡检开始时间!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="startTime"
          >
            <j-date
              placeholder="请选择计划巡检开始时间"
              v-model="model.startTime"
              :show-time="true"
              :disabled="formDisabled"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="计划巡检结束时间"
            :rules="[{ required: true, message: '请选择计划巡检开始时间!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="endTime"
          >
            <j-date
              placeholder="请选择计划巡检结束时间"
              v-model="model.endTime"
              :show-time="true"
              :disabled="formDisabled"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="提醒时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="warnTime">
            <a-input-number
              :disabled="formDisabled"
              v-model="model.warnTime"
              placeholder="请输入提醒时间"
              style="width: 48%"
            ></a-input-number
            >分钟
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="计划截止时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="cutOfTime">
            <j-date
              placeholder="请选择计划截止时间"
              v-model="model.cutOfTime"
              :show-time="true"
              :disabled="formDisabled"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="计划状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="planStatus">
            <a-select v-model="model.planStatus" :disabled="formDisabled" :showSearch="true" optionFilterProp="label">
              <a-select-option value="0" label="启用">启用</a-select-option>
              <a-select-option value="1" label="停用">停用</a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备范围" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentLimits">
            <a-input :disabled="formDisabled" v-model="model.equipmentLimits" placeholder="请输入设备范围"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="10">
          <a-form-model-item
            label="巡检人员"
            prop="executivePlanner"
            :labelCol="labelA"
            :wrapperCol="wrapperA"
            :rules="[{ required: true, message: '请选择巡检人员!' }]"
          >
            <j-select-multi-user
              v-model="model.executivePlanner"
              :query-config="selectUserQueryConfig"
              :disabled="formDisabled"
            /> </a-form-model-item
        ></a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">选择设备点位、巡检内容</div></a-col>
        <a-col :span="24">
          <a-alert
            type="info"
            showIcon
            message="点位巡检主要运用场景包含环境巡检、区域车间巡检等，是一种不属于设备的现场巡检"
          />
        </a-col>
        <a-col :span="24">
          <a-button
            style="margin-top:15px"
            v-if="!formDisabled"
            class="new-btn"
            @click="selectSparePart"
            type="primary"
          >
            <a-icon type="plus" />
            选择设备
          </a-button>
          <!-- <a-button
            style="margin-left:15px"
            v-if="!formDisabled"
            class="new-btn"
            @click="selectSparePart"
            type="primary"
          >
            <a-icon type="plus" />
            增加点位
          </a-button> -->
          <j-vxe-table
            ref="equipment"
            row-number
            row-selection
            keep-source
            :height="200"
            :loading="equipment.loading"
            :dataSource="equipment.dataSource"
            :columns="equipment.columns"
            :disabled="editDisabled"
            style="margin-top: 8px;"
          >
            <template v-slot:action="props">
              <!-- <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)"> -->
              <a @click="handleDL(props)">删除</a>
              <!-- </a-popconfirm> -->
            </template>
            <template v-slot:standard="props">
              <a @click="handleStandard(props, 0)"> 添加</a>
              <a @click="handleStandard(props, 1)"> 配置</a>
              <a-checkbox-group
                v-show="checkboxData"
                v-model="checkboxData"
                style="margin-left:10px"
                @change="checkboxChange"
                :options="checkboxOptions"
              />
            </template>
          </j-vxe-table>
        </a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">高级设置</div></a-col>
        <a-col :span="24">
          <a-alert
            type="info"
            showIcon
            style="margin-bottom:15px"
            message="主要运用场景包含节假日、休息日或休息时间段无需巡检的情况，当巡检计划时间处于跳过时间内，系统将默认跳过本次巡检任务"
          />
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="跳过时段1" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="skipTimeOne">
            <a-range-picker
              :disabled="formDisabled"
              @change="onChangeRangeDate1"
              format="YYYY-MM-DD"
              style="width:100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="跳过时段2" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="skipTimeOneEnd">
            <a-range-picker
              :disabled="formDisabled"
              @change="onChangeRangeDate2"
              format="YYYY-MM-DD"
              style="width:100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="跳过周末" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="skipWeekends">
            <a-checkbox-group v-model="model.skipWeekends" :disabled="formDisabled">
              <a-checkbox value="0">周六</a-checkbox>
              <a-checkbox value="1">周日</a-checkbox>
            </a-checkbox-group>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="跳过日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="skipDate">
            <a-date-picker
              :disabled="formDisabled"
              valueFormat="YYYY-MM-DD"
              v-model="model.skipDate"
              style="width:100%"
              allowClear
            />
          </a-form-model-item>
        </a-col>
      </a-row>
    </a-form-model>
    <!-- 选择明细 -->
    <j-modal
      title="选择明细"
      width="60%"
      :visible="spareVis"
      @ok="spareHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">保养明细</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="5">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.condition" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="保养部位">
                <j-dict-select-tag type="list" v-model="queryParam.spareType" dictCode="spare_type"></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="保养标准">
                <a-tree-select
                  v-model.trim="queryParam.storeHouse"
                  :treeData="treeData"
                  style="width: 100%"
                  :tree-data="treeData"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="spareTable"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="spareTable.columns"
        :dataSource="spareTable.dataSource"
        :pagination="spareTable.ipagination"
        :loading="spareTable.loading"
        :rowSelection="{ selectedRowKeys: spareTableSelectedRowKeys, onChange: spareTableOnSelectChange }"
        class="j-table-force-nowrap"
        @change="spareTableHandleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
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
                  :tree-data="treeData"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
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
        rowKey="id"
        :columns="equipmentTable.columns"
        :dataSource="equipmentTable.dataSource"
        :pagination="equipmentTable.ipagination"
        :loading="equipmentTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        class="j-table-force-nowrap"
        @change="handleEquipmentTableChange"
      >
      </a-table>
    </j-modal>
    <!-- 添加人员 -->
    <j-modal
      title="添加人员"
      width="60%"
      :visible="personnelVis"
      @ok="personnelOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchEquipment">
          <a-row :gutter="24">
            <a-col :span="6">
              <a-form-item label="姓名">
                <a-input v-model.trim="personnelParam.realname"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="执行班组">
                <j-dict-select-tag type="list" v-model="personnelParam.orgCodeTxt" dictCode="maintain_list" />
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="personnelParam.departIds_dictText"
                  style="width: 100%"
                  :tree-data="treeData"
                >
                </a-tree-select>
              </a-form-item>
            </a-col>
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchPersonnel" icon="search">查询</a-button>
                <a-button type="primary" @click="searchPersonnelReset" icon="reload" style="margin-left: 8px"
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
        rowKey="id"
        :columns="personnelTable.columns"
        :dataSource="personnelTable.dataSource"
        :pagination="personnelTable.ipagination"
        :loading="personnelTable.loading"
        :rowSelection="{ selectedRowKeys: selectedPersonnelRowKeys, onChange: onSelectChangePersonnel }"
        class="j-table-force-nowrap"
        @change="handlePersonnelTableChange"
      >
      </a-table>
    </j-modal>
    <!-- 增加标准 -->
    <j-modal
      title="增加标准"
      width="60%"
      :visible="standardVis"
      @ok="standardOk"
      @cancel="handleCancel"
      cancelText="关闭"
      okText="下一步"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchStandard">
          <a-row :gutter="24">
            <a-col :span="14">
              <!-- 查询区域 -->
              <a-form layout="inline" class="j-inline-form">
                <a-col :span="8">
                  <a-form-item label="设备类型">
                    <j-category-select
                      v-model="innerTableParam.equipmentType"
                      pcode="B03"
                      placeholder="请选择设备类型"
                    />
                  </a-form-item>
                </a-col>
                <a-col :span="8">
                  <a-form-item label="巡检项目">
                    <a-input v-model.trim="innerTableParam.standardName"> </a-input>
                  </a-form-item>
                </a-col>
                <a-col :span="4">
                  <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                    <a-button type="primary" icon="search">查询</a-button>
                    <a-button type="primary" icon="reload" style="margin-left: 8px">重置</a-button>
                  </span>
                </a-col>
                <!-- 按钮 -->
              </a-form>
              <a-table
                size="middle"
                bordered
                rowKey="id"
                :columns="innerTable.columns"
                :dataSource="innerTableData"
                :pagination="innerTable.ipagination"
                :loading="innerTable.loading"
                :scroll="{ y: 240 }"
                :rowSelection="{
                  innerTableRowKeys,
                  onChange: onSelectChangeInnerTable,
                  type: multiple ? 'checkbox' : 'radio'
                }"
                :customRow="customRowFn"
                @change="handleTableChange"
              >
              </a-table>
            </a-col>
            <a-col :span="10">
              <a-card :title="'已选'" :bordered="false" :head-style="{ padding: 0 }" :body-style="{ padding: 0 }">
                <a-table
                  size="middle"
                  rowKey="id"
                  bordered
                  :columns="selectedTable.columns"
                  :dataSource="selectedTable.dataSource"
                  :pagination="false"
                  :loading="selectedTable.loading"
                  :scroll="{ y: 240 }"
                >
                  <span slot="action" slot-scope="text, record, index">
                    <a @click="handleDeleteSelected(record, index)">删除</a>
                  </span>
                </a-table>
              </a-card></a-col
            >
          </a-row>
        </a-form>
      </div>
    </j-modal>
    <!-- 设置标准 -->
    <j-modal
      title="设置标准"
      width="60%"
      :visible="standardDetailVis"
      @ok="standardDetailOk"
      @cancel="standardDetailVis = false"
      cancelText="关闭"
    >
      <div class="table-page-search-wrapper">
        <a-table
          size="middle"
          :bordered="false"
          rowKey="id"
          :columns="standardDetailTable.columns"
          :data-source="standardDetailTableDataSource"
          :pagination="false"
          class="j-table-force-nowrap"
          :scroll="{ y: 500 }"
        >
          <template slot="required" slot-scope="text, record">
            <a-checkbox @change="$forceUpdate()" v-model="record.required"></a-checkbox>
          </template>
        </a-table>
      </div>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { loadCategoryData, queryDepartTreeList } from '@/api/api'
import { pushIfNotExist } from '@/utils/util'
import JUpload from '@/components/jeecg/JUpload'
import { cloneDeep } from 'lodash'
import { filterObj } from '@/utils/util'
import { FormTypes } from '@/utils/JEditableTableUtil'
import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

export default {
  name: 'TabTroubleFixReportForm',
  components: {
    JUpload
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
      isorterI: {
        column: 'createTime',
        orderBy: 'desc'
      },
      multiple: true,
      rowIndex: '',
      innerTable: {
        columns: [
          {
            title: '项目名称',
            align: 'center',
            dataIndex: 'standardName'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentType'
          }
        ],
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
      standardDetailTable: {
        dataSource: [],
        columns: [
          {
            title: '标准名称',
            align: 'center',
            dataIndex: 'standardName'
          },
          {
            title: '是否必填',
            align: 'center',
            dataIndex: 'required',
            scopedSlots: { customRender: 'required' }
          }
        ],
        loading: false
      },
      checkboxData: [],
      checkboxOptions: [],
      patrolExecutionMasterList: [],
      innerValue: [],
      innerTableData: [],
      standardDetailTableDataSource: [],
      selectedTable: {
        columns: [
          {
            title: '项目名称',
            align: 'center',
            dataIndex: 'standardName'
          },
          {
            title: '设备类型',
            align: 'center',
            dataIndex: 'equipmentType'
          },
          { title: '操作', dataIndex: 'action', align: 'center', width: 60, scopedSlots: { customRender: 'action' } }
        ],
        dataSource: [],
        loading: false
      },
      // 选择用户查询条件配置
      selectUserQueryConfig: [{ key: 'phone', label: '电话' }],

      fileList: [],
      // 选择明细
      spareVis: false,
      editDisabled: false,
      executeT: false,
      spareTable: {
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
            title: '设备类型',
            align: 'center',
            dataIndex: 'deviceType'
          },
          {
            title: '保养部位',
            align: 'center',
            dataIndex: 'keepPart'
          },
          {
            title: '保养标准',
            align: 'center',
            dataIndex: 'keepStandard'
          }
          // {
          //   title: '标准名称',
          //   align: 'center',
          //   dataIndex: 'orgCodeTxt'
          // }
        ]
      },
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
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      personnelTable: {
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
            title: '名称',
            align: 'center',
            dataIndex: 'realname'
          },
          {
            title: '性别',
            align: 'center',
            dataIndex: 'sex_dictText'
          },
          {
            title: '职务',
            align: 'center',
            dataIndex: 'post'
          },
          {
            title: '部门',
            align: 'center',
            dataIndex: 'orgCodeTxt'
          },
          {
            title: '执行班组',
            align: 'center',
            dataIndex: 'equipmentType',
            customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          }
        ]
      },
      equipment: {
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
            title: '设备编码',
            key: 'equipmentCode',
            type: FormTypes.normal
          },
          {
            title: '设备名称',
            key: 'equipmentName',
            type: FormTypes.normal
          },
          {
            title: '规格型号',
            key: 'equipmentModel',
            type: FormTypes.normal
          },
          {
            title: '巡检标准',
            key: 'standard',
            type: JVXETypes.slot,
            align: 'center',
            slotName: 'standard'
          },
          {
            title: '使用部门',
            key: 'depart',
            type: FormTypes.normal
          },
          {
            title: '设备类型',
            key: 'equipmentTypeName',
            type: FormTypes.normal
          }
        ]
      },
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
            title: '姓名',
            key: 'realname',
            type: FormTypes.normal
          },
          {
            title: '保养开始时间',
            key: 'equipmentName',
            type: FormTypes.datetime
          },
          {
            title: '保养完成时间',
            key: 'equipmentModel',
            type: FormTypes.datetime
          },
          {
            title: '保养用时',
            key: 'depart',
            type: FormTypes.normal,
            placeholder: '可修改单位为分钟数'
          },
          {
            title: '内容或步骤',
            key: 'equipmentType',
            type: FormTypes.input
          }
        ]
      },
      partTable: {
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
            title: '保养部位',
            key: 'keepPart',
            type: FormTypes.normal
          },
          {
            title: '保养标准',
            key: 'keepStandard',
            type: FormTypes.normal
          }
        ]
      },
      // end
      // 选择设备
      treeData: [],
      dictOptions: {},
      disableSubmit: false,
      loading: false,
      dataSource: [],
      selectedRowKeys: [],
      selectionRows: [],
      checkboxOptions: [],
      // 选择明细
      spareTableSelectedRowKeys: [],
      spareTableSelectionRows: [],
      // 添加人员弹窗
      selectedPersonnelRowKeys: [],
      selectionPersonnelRows: [],
      // 增加标准弹窗
      innerTableRowKeys: [],
      innerTableRows: [],

      queryParam: {},
      equipmentParam: {},
      personnelParam: {},
      innerTableParam: {},
      equipmentVis: false,
      personnelVis: false,
      standardVis: false,
      standardDetailVis: false,
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
      // end
      detailParam: {},
      model: {
        checkCycleTimeUnit: 'time1',
        planStatus: '0' //计划状态
        // keepType: 0, //保养类型：0：工单1：计划
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      labelA: {
        xs: { span: 24 },
        sm: { span: 4 }
      },
      wrapperA: {
        xs: { span: 24 },
        sm: { span: 20 }
      },
      confirmLoading: false,
      validatorRules: {
        checkCycle: [{ required: true, message: '请输入巡检周期!' }]
      },
      url: {
        list: '/checkout/pointpatrolstandard/page', //标准list
        findById: '/checkout/pointpatrolstandard/findById?id=', //标准
        add: '/checkplan/patrolInspectionPlan/add',
        edit: '/checkplan/patrolInspectionPlan/edit',
        executeKeepOrderInfo: '/realKeepOrder/tabKeepOrderReal/executeKeepOrderInfo', //执行保养 post
        partList: '/keep/tabKeepPart/list', //保养部位分页
        equipmentList: '/equipment/equipment/list', //设备管理分页
        userList: '/sys/user/list' //添加人员弹窗分页
      }
    }
  },
  watch: {
    value: {
      deep: true,
      immediate: true,
      handler(val) {
        this.innerValue = cloneDeep(val)
        this.selectedRowKeys = []
        this.valueWatchHandler(val)
        this.queryOptionsByValue(val)
      }
    },
    innerTableData: {
      deep: true,
      handler(val) {
        this.emitOptions(val)
        this.valueWatchHandler(this.innerValue)
      }
    },
    innerTableRowKeys: {
      immediate: true,
      deep: true,
      handler(val) {
        console.log('for：选取职务名称出现全选', val)
        //update--begin---for：选取职务名称出现全选 #1753-----
        if (this.innerValue) {
          this.innerValue.length = 0
        }
        //update--end---for：选取职务名称出现全选 #1753-----
        this.selectedTable.dataSource = val.map(key => {
          for (let data of this.innerTableData) {
            if (data.id === key) {
              pushIfNotExist(this.innerValue, data.id)
              return data
            }
          }
          for (let data of this.selectedTable.dataSource) {
            if (data.id === key) {
              pushIfNotExist(this.innerValue, data.id)
              return data
            }
          }
          console.warn('未找到选择的行信息，key：' + key)
          return {}
        })
        console.log('this.selectedTable.dataSource', this.selectedTable.dataSource)
      }
    }
  },
  computed: {
    // end
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
    this.initDictConfig()
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    searchStandard() {},
    emitOptions(dataSource, callback) {
      console.log('dataSource', dataSource)
      // dataSource.forEach(data => {
      // let key = data.id
      // this.dataSourceMap[key] = data
      // pushIfNotExist(this.options, { label: data[this.displayKey || this.valueKey], value: key }, 'value')
      // typeof callback === 'function' ? callback(data) : ''
      // })
    },
    valueWatchHandler(val) {
      console.log('innerTableData', this.innerTableData)
      val.forEach(item => {
        this.innerTableData.concat(this.selectedTable.dataSource).forEach(data => {
          if (data.id === item) {
            pushIfNotExist(this.innerTableRowKeys, data.id)
          }
        })
      })
    },
    queryOptionsByValue(value) {
      if (!value || value.length === 0) {
        return
      }
      // 判断options是否存在value，如果已存在数据就不再请求后台了
      let notExist = false
      for (let val of value) {
        let find = false
        for (let option of this.options) {
          if (val === option.value) {
            find = true
            break
          }
        }
        if (!find) {
          notExist = true
          break
        }
      }
      if (!notExist) return
      // getAction(this.valueUrl || this.listUrl, {
      //   // 这里最后加一个 , 的原因是因为无论如何都要使用 in 查询，防止后台进行了模糊匹配，导致查询结果不准确
      //   [this.valueKey]: value.join(',') + ',',
      //   pageNo: 1,
      //   pageSize: value.length
      // }).then(res => {
      //   if (res.success) {
      //     let dataSource = res.result
      //     if (!(dataSource instanceof Array)) {
      //       dataSource = res.result.records
      //     }
      //     this.emitOptions(dataSource, data => {
      //       pushIfNotExist(this.innerValue, data[this.valueKey])
      //       pushIfNotExist(this.innerTableRowKeys, data[this.rowKey])
      //       pushIfNotExist(this.selectedTable.dataSource, data, this.rowKey)
      //     })
      //   }
      // })
    },
    customRowFn(record) {
      return {
        on: {
          click: () => {
            let key = record.id
            if (!this.multiple) {
              this.innerTableRowKeys = key
              this.selectedTable.dataSource = record
            } else {
              let index = this.innerTableRowKeys.indexOf(key)
              if (index === -1) {
                this.innerTableRowKeys.push(key)
                this.selectedTable.dataSource.push(record)
              } else {
                this.handleDeleteSelected(record, index)
              }
            }
          }
        }
      }
    },
    /** 删除已选择的 */
    handleDeleteSelected(record, index) {
      console.log('index', index)
      this.innerTableRowKeys.splice(this.innerTableRowKeys.indexOf(record.id), 1)
      //update--begin-------for：JSelectBizComponent组件切换页数值问题------
      this.selectedTable.dataSource.splice(this.selectedTable.dataSource.indexOf(record), 1)
      this.innerValue.splice(this.innerValue.indexOf(record.id), 1)
      console.log('this.innerValue', this.innerValue)
      console.log('this.innerTableRowKeys:', this.innerTableRowKeys)
      console.log('this.selectedTable.dataSource:', this.selectedTable.dataSource)
      //update--begin------for：JSelectBizComponent组件切换页数值问题------
    },
    onChangeRangeDate1(value, dateString) {
      this.model.skipTimeOne = dateString[0]
      this.model.skipTimeOneEnd = dateString[1]
    },
    onChangeRangeDate2(value, dateString) {
      this.model.skipTimeTwo = dateString[0]
      this.model.skipTimeTwoEnd = dateString[1]
    },
    handleDL(props) {
      // console.log('props', props)
      // 调用删除方法
      props.target.removeRows(props.row)
      this.equipment.dataSource.splice(props.rowIndex, 1)
    },
    handleStandard(val, type) {
      this.rowIndex = val.rowIndex
      // console.log('this.rowIndex', this.rowIndex)
      if (type == 0) {
        var params = this.getStandard() //查询条件
        getAction(this.url.list, params).then(res => {
          if (res.code == 200) {
            this.innerTableData = res.data.records || res.data
            if (res.data.total) {
              this.innerTable.ipagination.total = res.data.total
            } else {
              this.innerTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.msg)
          }
        })
        this.standardVis = true
      } else {
        this.standardDetailVis = true
      }
    },
    getStandard() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.innerTableParam, this.isorterI, this.filters)
      param.status = '0'
      param.pageNum = this.innerTable.ipagination.current
      param.pageSize = this.innerTable.ipagination.pageSize
      return filterObj(param)
    },
    standardOk() {
      console.log('this.selectedTable.dataSource', this.selectedTable.dataSource)
      this.selectedTable.dataSource.forEach(item => {
        this.standardEdit(item.id)
      })
    },
    standardNameChange(val) {
      console.log('val', val)
    },
    standardEdit(id) {
      var that = this
      getAction(that.url.findById + id).then(res => {
        if (res.code == 200) {
          let arr = res.data
          arr.inspectionstandardList.forEach(item => {
            item.standardName = item.name
          })
          arr.children = arr.inspectionstandardList
          that.standardDetailTableDataSource.push(arr)
        }
      })
      console.log('that.standardDetailTableDataSource', that.standardDetailTableDataSource)
      that.innerTableData = []
      that.innerTableRowKeys = []
      that.innerTableRows = []
      that.standardVis = false
      that.innerTableParam = {}
      that.standardDetailVis = true
    },
    checkboxChange() {
      this.checkboxOptions = Object.assign(this.checkboxData)
      this.$forceUpdate()
    },
    standardDetailOk() {
      var that = this
      // var data = {}
      console.log('that.equipment.dataSource[that.rowIndex]', that.equipment.dataSource[that.rowIndex])
      // that.equipment.dataSource[that.rowIndex].patrolExecutionMasterList = this.standardDetailTableDataSource.map(
      //   val => {
      //     data.id = val.id
      //     data.standardName = val.standardName
      //     data.required = val.required
      //     data.patrolExecutionSecondaryList = val.children
      //     return data
      //   }
      // )
      // this.standardDetailTableDataSource.forEach(item => {
      //   let pm = {
      //     id: item.id,
      //     standardName: item.standardName,
      //     required: item.required,
      //     patrolExecutionSecondaryList: item.children
      //   }
      //   that.equipment.dataSource[that.rowIndex].patrolExecutionMasterList.push(pm)
      // })
      console.log('that.equipment.dataSource', that.equipment.dataSource)
      that.checkboxOptions = that.standardDetailTableDataSource.map(val => {
        return val.standardName
      })
      that.checkboxData = that.standardDetailTableDataSource.map(val => {
        return val.standardName
      })
      that.standardDetailVis = false
    },
    // 选择设备
    initDictConfig() {
      loadCategoryData({ code: '' }).then(res => {
        if (res.success) {
          console.log('选择设备.equipmentType', res)
          this.$set(this.dictOptions, 'equipmentType', res.result)
        }
      })
    },
    // 保养分页表
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    spareTableOnSelectChange(spareTableSelectedRowKeys, spareTableSelectionRows) {
      this.spareTableSelectedRowKeys = spareTableSelectedRowKeys
      this.spareTableSelectionRows = spareTableSelectionRows
    },
    onSelectChangePersonnel(selectedPersonnelRowKeys, selectionPersonnelRows) {
      this.selectedPersonnelRowKeys = selectedPersonnelRowKeys
      this.selectionPersonnelRows = selectionPersonnelRows
    },
    onSelectChangeInnerTable(innerTableRowKeys, innerTableRows) {
      this.innerTableRowKeys = innerTableRowKeys
      this.innerTableRows = innerTableRows
    },
    spareTableHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.spareTable.ipagination = pagination
      this.loadData()
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.loadData()
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
    handlePersonnelTableChange(pagination, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.personnelTable.ipagination = pagination
      this.personnelLoad()
    },
    handleOk() {
      this.equipment.dataSource = this.selectionRows
      this.equipmentVis = false
    },
    personnelOk() {
      this.personnel.dataSource = this.selectionPersonnelRows
      this.personnelVis = false
    },
    handleValueChange(event) {
      console.log('handleValueChange.event: ', event)
    },
    spareHandleOk() {
      this.partTable.dataSource = this.spareTableSelectionRows
      this.spareVis = false
    },
    handleCancel() {
      this.equipmentVis = false
      this.equipmentVis = false
      this.spareVis = false
      this.personnelVis = false
      this.standardVis = false
      this.standardDetailVis = false
    },
    searchQuery() {
      this.loadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    searchReset() {
      this.queryParam = {}
      this.loadData(1)
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction('/equipment/equipment/newTroubleList', params)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
    },
    // end
    // 设备管理
    selectSparePart() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    searchEquipment() {
      this.equipmentLoad(1)
      this.equipmentSelectedRowKeys = []
      this.equipmentSelectionRows = []
    },
    searchEquipmentReset() {
      this.equipmentParam = {}
      this.equipmentLoad(1)
    },
    searchPersonnel() {
      this.personnelLoad(1)
      // 点击查询清空列表选中行
      this.selectedPersonnelRowKeys = []
      this.selectionPersonnelRows = []
    },
    searchPersonnelReset() {},
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
    personnelLoad(arg) {
      if (!this.url.equipmentList) {
        this.$message.error('请设置url.equipmentList属性!')
        return
      }
      if (arg === 1) {
        this.personnelTable.ipagination.current = 1
      }
      var params = Object.assign(this.personnelParam)
      params.pageNo = this.personnelTable.ipagination.current
      params.pageSize = this.personnelTable.ipagination.pageSize
      this.personnelTable.loading = true
      getAction(this.url.userList, params, this.isorter)
        .then(res => {
          if (res.success) {
            this.personnelTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.personnelTable.ipagination.total = res.result.total
            } else {
              this.personnelTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.personnelTable.loading = false
        })
    },
    // end
    selectPartTable() {
      this.spareData()
      this.spareVis = true
    },
    // 保养部位-分页查询
    spareData(arg) {
      if (!this.url.partList) {
        this.$message.error('请设置url.partList属性!')
        return
      }
      if (arg === 1) {
        this.spareTable.ipagination.current = 1
      }
      var params = this.getQueryParams()
      this.spareTable.loading = true
      getAction(this.url.partList, params)
        .then(res => {
          if (res.success) {
            this.spareTable.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.spareTable.ipagination.total = res.result.total
            } else {
              this.spareTable.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.spareTable.loading = false
        })
    },
    loadData(arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams()
      this.loading = true
      getAction(this.url.list, params)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.ipagination.total = res.result.total
            } else {
              this.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.loading = false
        })
    },
    getQueryParams() {
      //获取查询条件
      let sqp = {}
      if (this.superQueryParams) {
        sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
      // param.field = this.getQueryField()
      param.pageNo = this.ipagination.current
      param.pageSize = this.ipagination.pageSize
      return filterObj(param)
    },
    getQueryField() {
      //TODO 字段权限控制
      var str = 'id,'
      this.columns.forEach(function(value) {
        str += ',' + value.dataIndex
      })
      return str
    },
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
      this.editDisabled = true
    },
    edit(record) {
      getAction('/checkplan/patrolInspectionPlan/queryById?id=' + record.id).then(res => {
        this.model = Object.assign({}, res.result)
        console.log('this.model', this.model)
        this.equipment.dataSource = this.model.standards
        // this.patrolExecutionMasterList = this.model.patrolExecutionMasterList
      })
      this.visible = true
      this.editDisabled = false
    },
    // 执行保养
    execute(record) {
      this.searchList(record)
      console.log('execute', record)
      this.executeT = true
      setTimeout(() => {
        this.model.keepProcessStatus = '3'
        this.visible = true
        this.editDisabled = false
      }, 500)
    },
    searchList(val) {
      // 根据id查询保养部位、设备管理
      getAction('/realKeepOrder/tabKeepOrderReal/queryById?id=' + val.id).then(res => {
        if (res.success) {
          this.model = res.result.order
          if (res.result.equipment) {
            this.equipment.dataSource.push(res.result.equipment)
          }
          this.$forceUpdate()
          // console.log('  根据id查询保养部位、设备管理', this.equipment.dataSource)
        } else {
          this.$message.warning(res.message)
        }
      })
    },

    submitForm() {
      const that = this
      let parm = that.model
      that.equipment.dataSource.forEach(item => {
        item.equipmentId = item.id
        delete item.id
      })
      parm.standards = that.equipment.dataSource
      console.log('parm', parm)
      // 触发表单验证
      that.$refs.form.validate(valid => {
        if (valid) {
          if (this.model.checkCycle) {
            if (this.standardDetailTableDataSource.length != 0) {
              that.confirmLoading = true
              let httpurl = ''
              let method = ''
              if (!that.model.id) {
                httpurl += that.url.add
                method = 'post'
              } else {
                if (that.executeT) {
                  httpurl += that.url.executeKeepOrderInfo
                  method = 'post'
                } else {
                  httpurl += that.url.edit
                  method = 'put'
                }
              }
              httpAction(httpurl, parm, method)
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
              that.$message.warning('部分设备未选择巡检项!')
            }
          } else {
            that.$message.warning('请输入巡检周期!')
          }
        }
      })
    },
    // 添加人员
    handlePersonnel() {
      this.personnelLoad()
      this.personnelVis = true
    }
    // end
  }
}
</script>
<style lang="less" scoped>
::v-deep {
  .ant-card-head-title {
    padding: 0;
  }
  .ant-modal-body {
    margin: 0;
  }
  .ant-modal-body {
    margin-bottom: 0 !important;
  }
  // .ant-spin-container {
  //   margin-top: 50px;
  // }
}
</style>
