<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model">
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">处理情况</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="保养班组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="keepUsers">
            <!-- <j-dict-select-tag
              type="list"
              v-model="model.keepUsers"
              dictCode="maintain_list"
              :disabled="formDisabled"
              placeholder="请选择保养班组"
            /> -->
            <a-tree-select
              v-model="model.keepUsers"
              style="width: 100%"
              :tree-data="treeData"
              :disabled="formDisabled"
              show-search
              tree-default-expand-all
              tree-node-filter-prop="title"
              placeholder="请选择保养班组"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="保养状态"
            :labelCol="labelCol"
            :rules="[{ required: true, message: '请选择保养状态!' }]"
            :wrapperCol="wrapperCol"
            prop="keepProcessStatus"
          >
            <a-select
              v-model="model.keepProcessStatus"
              class="code-mode-select"
              :disabled="formDisabled"
              placeholder="请选择保养状态"
              :showSearch="true"
              optionFilterProp="label"
            >
              <a-select-option value="3" label="保养中">保养中</a-select-option>
              <a-select-option value="5" label="已完成">已完成</a-select-option>
              <a-select-option v-if="formDisabled" value="6" label="已完成">已完成</a-select-option>
              <!-- 待验证5已完成6 -->
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="保养级别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="keepLevel">
            <j-dict-select-tag
              type="list"
              dictCode="keep_level"
              :disabled="formDisabled"
              v-model.trim="model.keepLevel"
            ></j-dict-select-tag>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="外委单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outCompany">
            <!-- <a-input
              :disabled="formDisabled"
              v-model.trim="model.outsourceUnits"
              placeholder="请输入外委单位"
            ></a-input> -->
            <a-select
              :disabled="formDisabled"
              v-model="model.outsourceUnits"
              :showSearch="true"
              optionFilterProp="label"
              placeholder="回车可搜索"
              style="width: 100%"
            >
              <a-select-option
                v-for="(item, index) in companyOptions"
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
          <a-form-model-item
            label="保养开始时间"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="startTime"
            :rules="[{ required: true, message: '请选择保养开始时间!' }]"
          >
            <j-date
              placeholder="请选择保养开始时间"
              :disabled="formDisabled"
              v-model.trim="model.startTime"
              :show-time="true"
              @change="changeTime"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
            <!-- <a-date-picker
              showTime
              @change="changeTime"
              style="width: 100%"
              :disabled="formDisabled"
              placeholder="请选择保养开始时间"
              v-model.trim="model.startTime"
              format="YYYY-MM-DD HH:mm:ss"
            /> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="保养结束时间"
            :rules="[{ required: true, message: '请选择保养结束时间!' }]"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            prop="endTime"
          >
            <j-date
              placeholder="请选择保养结束时间"
              :disabled="formDisabled"
              v-model.trim="model.endTime"
              @change="changeTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="保养用时" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input :disabled="formDisabled" v-model.trim="model.keepHour" style="width: 40%;"> </a-input>小时
            <a-input :disabled="formDisabled" v-model.trim="model.keepMin" style="width: 40%;"> </a-input>分
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="保养费用" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input :disabled="formDisabled" v-model.trim="model.keepPrice" placeholder="请输入保养费用"> </a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="是否停机" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-radio-group v-model="model.isOff" :disabled="formDisabled">
              <a-radio value="0">否</a-radio>
              <a-radio value="1">是</a-radio>
            </a-radio-group>
          </a-form-model-item>
        </a-col>
        <a-col v-if="model.isOff != '1' ? false : true" :span="8">
          <a-form-model-item label="停机时长" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input v-model.trim="model.offTime" placeholder="请输入停机时长"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="经验代码" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input :disabled="formDisabled" v-model="model.exCode" placeholder="请输入经验代码"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="工作描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input
              :disabled="formDisabled"
              v-model.trim="model.describeText"
              type="textarea"
              placeholder="请输入工作描述"
            >
            </a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">设备信息</div></a-col>
        <a-col :span="24">
          <a-button
            :disabled="formDisabled"
            v-if="!formDisabled && !executeT"
            class="new-btn"
            @click="selectSparePart"
            type="primary"
          >
            <a-icon type="plus" />
            选择设备
          </a-button>
          <j-vxe-table
            ref="equipment"
            row-number
            row-selection
            keep-source
            :height="200"
            :loading="equipment.loading"
            :dataSource="equipment.dataSource"
            :columns="equipment.columns"
            :disabled="formDisabled"
            style="margin-top: 8px;"
          >
            <template v-slot:action="props" v-if="!formDisabled">
              <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
                <a>删除</a>
              </a-popconfirm>
            </template></j-vxe-table
          >
        </a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">保养工作量</div></a-col>
        <a-col :span="24">
          <a-button
            class="new-btn"
            type="primary"
            :disabled="formDisabled"
            v-if="!formDisabled"
            @click="handlePersonnel"
          >
            <a-icon type="plus" />
            添加人员
          </a-button>
          <j-vxe-table
            ref="personnel"
            row-number
            row-selection
            keep-source
            :height="200"
            :disabled="formDisabled"
            :loading="personnel.loading"
            :dataSource="personnel.dataSource"
            :columns="personnel.columns"
            @valueChange="handleValueChange"
            style="margin-top: 8px;"
          >
            <template v-slot:action="props">
              <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
                <a>删除</a>
              </a-popconfirm>
            </template></j-vxe-table
          ></a-col
        >
      </a-row>
    </a-form-model>
    <div class="title" style="margin:10px 0">保养部位</div>
    <a-button
      style="margin-top:10px"
      v-if="!formDisabled && !executeT"
      class="new-btn"
      @click="selectPartTable"
      type="primary"
    >
      <a-icon type="search" />
      选择明细
    </a-button>
    <j-vxe-table
      ref="partTable"
      row-number
      row-selection
      keep-source
      :height="200"
      :loading="partTable.loading"
      :dataSource="partTable.dataSource"
      :columns="partTable.columns"
      style="margin-top: 8px;"
    >
      <template v-slot:action="props">
        <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
          <a>删除</a>
        </a-popconfirm>
      </template></j-vxe-table
    >

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
            <a-col :span="6">
              <a-form-item label="设备类别">
                <j-category-select v-model="queryParam.deviceType" pcode="B03" />
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="保养部位">
                <a-input v-model="queryParam.keepPart"></a-input>
                <!-- <a-select show-search v-model="queryParam.keepPart" placeholder="请输入" style="width: 100%">
                  <a-select-option v-for="item in keepPartData" :key="item.id" :value="item.keepPart">
                    {{ item.keepPart }}
                  </a-select-option>
                </a-select> -->
              </a-form-item>
            </a-col>
            <a-col :span="6">
              <a-form-item label="保养标准">
                <a-input v-model="queryParam.keepStandard"></a-input>
                <!-- <a-select show-search v-model="queryParam.keepStandard" placeholder="请输入" style="width: 100%">
                  <a-select-option v-for="item in keepPartData" :key="item.id" :value="item.keepStandard">
                    {{ item.keepStandard }}
                  </a-select-option>
                </a-select> -->
              </a-form-item>
            </a-col>
            <!-- <a-col :span="5">
              <a-form-item label="标准名称">
                <a-tree-select
                  v-model.trim="queryParam.storeHouse"
                  :treeData="treeData"
                  style="width: 100%"
                  :tree-data="treeData"
                >
                </a-tree-select>
              </a-form-item>
            </a-col> -->
            <a-col :span="4">
              <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchSpare" icon="search">查询</a-button>
                <a-button type="primary" @click="searchSpareReset" icon="reload" style="margin-left: 8px"
                  >重置</a-button
                >
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
                  :treeData="treeData"
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
                <a-input v-model.trim="personnelParam.username"></a-input>
              </a-form-item>
            </a-col>
            <!-- <a-col :span="6">
              <a-form-item label="执行班组">
                <j-dict-select-tag type="list" v-model="personnelParam.orgCodeTxt" dictCode="maintain_list" />
              </a-form-item>
            </a-col> -->
            <a-col :span="6">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="personnelParam.departId"
                  show-search
                  tree-default-expand-all
                  tree-node-filter-prop="title"
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
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction, postAction } from '@/api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'
import { axios } from '@/utils/request'
import { loadCategoryData, queryDepartTreeList, getUserList } from '@/api/api'
import JUpload from '@/components/jeecg/JUpload'
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
      keepPartData: [],
      companyOptions: [],
      keepStatusOptions: [
        {
          label: '待保养',
          value: '3'
        },
        {
          label: '保养中',
          value: '4'
        },
        {
          label: '待验证',
          value: '5'
        },
        {
          label: '已完成',
          value: '6'
        },
        {
          label: '已逾期',
          value: '7'
        }
      ],
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
            dataIndex: 'username'
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
          }
          // {
          //   title: '执行班组',
          //   align: 'center',
          //   dataIndex: 'equipmentType',
          //   customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
          // }
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
            disabled: this.formDisabled,
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
            title: '使用部门',
            key: 'depart',
            type: FormTypes.normal
          },
          {
            title: '设备类型',
            key: 'equipmentTypeName',
            // customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : ''),
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
            disabled: this.formDisabled,
            align: 'center',
            slotName: 'action'
          },
          {
            title: '姓名',
            key: 'userName',
            type: FormTypes.normal
          },
          {
            title: '保养开始时间',
            key: 'keepStartTime',
            type: FormTypes.datetime
          },
          {
            title: '保养完成时间',
            key: 'keepEndTime',
            type: FormTypes.datetime
          },
          {
            title: '保养用时',
            key: 'keepUseTime',
            width: 147,
            type: FormTypes.normal,
            placeholder: '可修改单位为分钟数'
          },
          {
            title: '内容或步骤',
            key: 'content',
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
            disabled: this.formDisabled,
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
      // 选择明细
      spareTableSelectedRowKeys: [],
      spareTableSelectionRows: [],
      // 添加人员弹窗
      selectedPersonnelRowKeys: [],
      selectionPersonnelRows: [],

      queryParam: {},
      equipmentParam: {},
      personnelParam: {},
      equipmentVis: false,
      personnelVis: false,
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
      cycleOption: [
        {
          label: '天',
          value: 'time1'
        },
        {
          label: '月',
          value: 'time2'
        },
        {
          label: '季',
          value: 'time3'
        }
      ],
      detailParam: {},
      model: {
        // partRelationList: [],
        periodType: 'time1',
        circleWay: '1',
        // keepType: 0, //保养类型：0：工单1：计划
        deviceId: ''
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 6 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      url: {
        add: '/realKeepOrder/tabKeepOrderReal/add',
        edit: '/realKeepOrder/tabKeepOrderReal/edit',
        executeKeepOrderInfo: '/realKeepOrder/tabKeepOrderReal/executeKeepOrderInfo', //执行保养 post
        queryById: '/troublereport/tabTroubleFixReport/queryById',
        partList: '/keep/tabKeepPart/list', //保养部位分页
        equipmentList: '/equipment/equipment/list', //设备管理分页
        // userList: '/sys/user/userList' //添加人员弹窗分页
        userList: '/sys/user/list' //添加人员弹窗分页
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
    // 往来单位
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        console.log('res.供应商', res.result)
        this.companyOptions = arr
      }
    })
  },
  methods: {
    handleValueChange(event) {
      let row = event.row
      var oDate1 = new Date(row.keepStartTime)
      var oDate2 = new Date(row.keepEndTime)
      if (oDate1.getTime() > oDate2.getTime()) {
        row.keepStartTime = ''
        this.$message.warning('开始时间小于结束时间请重新选择！')
      } else {
        let data = this.getHoursDiff(row.keepStartTime, row.keepEndTime)
        var h = parseInt(data)
        var m = parseInt((data - h) * 60)
        row.keepUseTime = h + '小时' + m + '分钟'
      }
    },
    changeTime() {
      if (this.model.startTime && this.model.endTime) {
        this.compareDate(this.model.startTime, this.model.endTime)
      }
    },
    // 比较时间大小
    compareDate(date1, date2) {
      var oDate1 = new Date(date1)
      var oDate2 = new Date(date2)
      if (oDate1.getTime() > oDate2.getTime()) {
        this.model.startTime = ''
        this.$message.warning('开始时间小于结束时间请重新选择！')
      }
    },
    getHoursDiff(dt1, dt2) {
      if (typeof dt1 == 'string') {
        if (dt1 != null) {
          dt1 = new Date(dt1.replace(/-/, '/'))
        }
        if (dt2 != null) {
          dt2 = new Date(dt2.replace(/-/, '/'))
        }
      }
      var res = dt2 - dt1
      if (isNaN(res)) throw Error('invalid dates arguments')
      return res / (1000 * 60 * 60)
    },

    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
      // 选择设备
      this.selectedRowKeys = []
      this.selectionRows = []
      // 保养工作量
      this.selectedPersonnelRowKeys = []
      this.selectionPersonnelRows = []
      // 明细
      this.spareTableSelectedRowKeys = []
      this.spareTableSelectionRows = []
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
    spareTableHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.spareTable.ipagination = pagination
      this.spareData()
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
      this.personnel.dataSource = []
      console.log('this.selectionPersonnelRows', this.selectionPersonnelRows)
      this.selectionPersonnelRows.forEach(item => {
        this.personnel.dataSource.push(item)
        item.userName = item.username
      })
      this.personnel.dataSource = this.selectionPersonnelRows
      this.personnelVis = false
    },
    spareHandleOk() {
      this.partTable.dataSource = this.spareTableSelectionRows
      this.handleCancel()
    },
    handleCancel() {
      this.equipmentVis = false
      this.equipmentVis = false
      this.personnelParam = {}
      this.queryParam = {}
      this.spareVis = false
      this.personnelVis = false
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
      // getAction('/equipment/equipment/list').then(res => {
      //   if (res.success) {
      //     this.equipmentTable.dataSource = res.result.records || []
      //     console.log('  this.dataSource', this.dataSource)
      //   }
      // })
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
    searchPersonnelReset() {
      this.personnelParam = {}
      this.personnelLoad()
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
    searchSpare() {
      this.spareData(1)
      this.spareTableSelectedRowKeys = []
      this.spareTableSelectionRows = []
    },
    searchSpareReset() {
      this.queryParam = {}
      this.spareData(1)
    },
    selectPartTable() {
      this.spareData()
      // this.keepList()
      this.spareVis = true
    },
    keepList() {
      // 保养工单，查询保养部位，保养标准
      getAction('/keep/tabKeepPart/list', { pageNo: 1, pageSize: 999999 }).then(res => {
        if (res.success) {
          let arr = [...res.result.records]
          console.log('res.result', res.result.records)
          this.keepPartData = arr //部位
          console.log(' this.keepPartData', this.keepPartData)
        }
      })
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
      var params = {
        deviceType: this.queryParam.deviceType,
        keepPart: this.queryParam.keepPart,
        keepStandard: this.queryParam.keepStandard,
        pageNo: this.spareTable.ipagination.current,
        pageSize: this.spareTable.ipagination.pageSize
      }
      console.log('params', params)
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
      this.model.isOff = '0'
      this.visible = true
      this.editDisabled = true
    },
    edit(record) {
      // this.model = Object.assign({}, record)
      this.searchList(record)
      this.visible = true
      this.editDisabled = true
      this.formDisabled
      console.log('this.formDisabled', this.formDisabled)
    },
    // 执行保养
    execute(record) {
      this.searchList(record)
      this.executeT = true
      setTimeout(() => {
        this.model.keepStatus = '3'
        this.visible = true
        this.editDisabled = false
      }, 500)
    },
    searchList(val) {
      // 根据id查询保养部位、设备管理
      getAction('/realKeepOrder/tabKeepOrderReal/queryById?id=' + val.id).then(res => {
        if (res.success) {
          // 保养工单
          this.equipment.dataSource.push(res.result.equipment)
          // 保养部位
          res.result.partRelationList.forEach(item => {
            this.partTable.dataSource.push(item)
          })
          // 保养工作量
          res.result.keepDetailUsers.forEach(item => {
            this.personnel.dataSource.push(item)
          })
          // 设备信息

          this.model = res.result.order
          this.$forceUpdate()
          // console.log('  根据id查询保养部位、设备管理', this.equipment.dataSource)
          // console.log('  根据id查询保养部位、设备管理', this.partTable.dataSource)
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    submitForm() {
      const that = this
      console.log('that.equipment.dataSource', that.equipment.dataSource)
      if (that.equipment.dataSource.length != 0) {
        if (that.model.deviceId) {
        } else {
          that.equipment.dataSource.forEach(val => {
            that.model.deviceId += val.id + ','
          })
          //
          that.model.deviceId =
            that.model.deviceId.substring(that.model.deviceId.length - 1) == ','
              ? that.model.deviceId.substring(0, that.model.deviceId.length - 1)
              : that.model.deviceId
        }
        let parm = {
          order: that.model,
          partRelationList: that.partTable.dataSource,
          keepDetailUsers: that.personnel.dataSource
        }
        // 触发表单验证
        that.$refs.form.validate(valid => {
          if (valid) {
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
          }
        })
      } else {
        that.$message.warning('请至少选择一个设备添加计划!')
      }
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
.ant-modal-body {
  margin-bottom: 0 !important;
}
</style>
