<template>
  <a-spin :spinning="confirmLoading">
    <a-form-model ref="form" :model="model">
      <a-row>
        <a-col :span="24"><div class="title" style="margin:10px 0">设备信息</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="维修单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="repairOrderNo">
            <a-input :disabled="formDisabled" v-model.trim="model.repairOrderNo" placeholder="不填可自动生成"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备编码" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentCode">
            <a-input :disabled="true" v-model.trim="model.equipmentCode" placeholder="请输入设备编码"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item
            label="设备名称"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            :rules="[{ required: true, message: '请输入设备名称!' }]"
          >
            <a-input :disabled="true" v-model.trim="model.equipmentName" placeholder="请输入设备名称"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="设备类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentType">
            <j-category-select
              :disabled="true"
              v-model.trim="model.equipmentType"
              pcode="B03"
              placeholder="请选择设备类型"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="规格型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentModel">
            <a-input :disabled="true" v-model.trim="model.equipmentModel" placeholder="请输入规格型号"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="所在部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="address">
            <a-tree-select
              :disabled="true"
              v-model.trim="model.address"
              :treeData="treeData"
              style="width: 100%"
              :tree-data="treeData"
              placeholder="请选择所在部门"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8" style="margin-left:7%">
          <a-button @click="selectEquipment" :disabled="formDisabled" type="primary">
            <a-icon type="search" /> 选择设备
          </a-button>
        </a-col>
        <a-col :span="24"><div class="title" style="margin:10px 0">故障信息</div></a-col>

        <a-col :span="8">
          <a-form-model-item label="发生时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleTime">
            <j-date
              :disabled="formDisabled"
              placeholder="请选择发生时间"
              v-model.trim="model.troubleTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="操作人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operator">
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.operator"
              placeholder="请选择操作人"
              dictCode="sys_user,realname,id"
            />
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.operator" placeholder="请输入操作人"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障部位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troublePart">
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.troublePart" placeholder="请输入故障部位"></a-input> -->
            <j-dict-select-tag
              :disabled="formDisabled"
              v-model="model.troublePart"
              placeholder="请选择故障部位"
              dictCode="trouble_location"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleType">
            <j-category-select
              :disabled="formDisabled"
              v-model.trim="model.troubleType"
              pcode="B05"
              placeholder="请选择故障类别"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障等级" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleLevel">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model.trim="model.troubleLevel"
              dictCode="failure_level"
              placeholder="请选择故障等级"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="故障描述" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input
              type="textarea"
              :disabled="formDisabled"
              v-model.trim="model.troubleDescription"
              placeholder="请输入故障描述"
            ></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="24"> <div class="title">处理情况</div></a-col>
        <a-col :span="8">
          <a-form-model-item label="故障原因" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="troubleReason">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.troubleReason"
              dictCode="failure_cause"
              placeholder="请选择故障原因"
            />
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.troubleReason" placeholder="请输入故障原因"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修组" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixTeam">
            <!-- <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.fixTeam"
              dictCode="maintain_list"
              placeholder="请选择维修组"
            />     -->
            <a-tree-select
              :disabled="formDisabled"
              v-model="model.fixTeam"
              style="width: 100%"
              :tree-data="treeData"
              placeholder="请选择维修组"
            >
            </a-tree-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixStatus">
            <!-- <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.fixStatus"
              dictCode="fix_work_order_status"
              placeholder="请选择维修状态"
            />  -->
            <a-select
              :disabled="formDisabled"
              v-model="model.fixStatus"
              class="code-mode-select"
              :showSearch="true"
              optionFilterProp="label"
              placeholder="请选择维修状态"
            >
              <a-select-option
                v-for="(item, index) in keepStatusOptions"
                :key="index"
                :label="item.label"
                :value="item.value"
              >
                {{ item.label }}
              </a-select-option>
            </a-select>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修级别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixLevel">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.fixLevel"
              dictCode="service_level"
              placeholder="请选择维修级别"
            />
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.fixLevel" placeholder="请输入维修级别"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修类别" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixType">
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.fixType" placeholder="请输入维修类别"></a-input> -->
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.fixType"
              dictCode="service_category"
              placeholder="请选择维修类别"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="紧急程度" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="urgency">
            <j-dict-select-tag
              type="list"
              :disabled="formDisabled"
              v-model="model.urgency"
              dictCode="emergency_degree"
              placeholder="请选择紧急程度"
            />
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.urgency" placeholder="请输入紧急程度"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="是否停机" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stop">
            <a-radio-group v-model="model.stop" :disabled="formDisabled">
              <a-radio value="0">否</a-radio>
              <a-radio value="1">是</a-radio>
            </a-radio-group>
            <!-- <a-input :disabled="formDisabled" v-model.trim="model.stop" placeholder="请输入是否停机"></a-input> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="8" v-if="model.stop == '1'">
          <a-form-model-item label="停机时长" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="stopTime">
            <a-input :disabled="formDisabled" v-model.trim="model.stopTime" placeholder="请输入停机时长"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="startTime">
            <j-date
              :disabled="formDisabled"
              placeholder="请选择维修开始时间"
              v-model.trim="model.startTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endTime">
            <j-date
              :disabled="formDisabled"
              placeholder="请选择维修结束时间"
              v-model.trim="model.endTime"
              :show-time="true"
              date-format="YYYY-MM-DD HH:mm:ss"
              style="width: 100%"
              @change="changeTime"
            />
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修用时（时）" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input
              :disabled="formDisabled"
              style="width: 37%;margin-right: 10px"
              v-model.number="model.fixHour"
              placeholder="请输入小时"
            ></a-input
            >小时
            <a-input
              :disabled="formDisabled"
              style="width: 38%;margin-right: 20px"
              v-model.number="model.fixMin"
              placeholder="请输入分"
            ></a-input
            >分
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="维修费用" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fixSpend">
            <a-input :disabled="formDisabled" v-model.trim="model.fixSpend" placeholder="请输入维修费用"></a-input>
          </a-form-model-item>
        </a-col>
        <a-col :span="8">
          <a-form-model-item label="外委单位" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="outCompany">
            <a-select
              :disabled="formDisabled"
              :showSearch="true"
              v-model="model.outCompany"
              placeholder="回车可搜索"
              style="width: 100%"
            >
              <a-select-option v-for="(item, index) in companyOptions" :key="index" :value="item.id">
                {{ item.companyName }}
              </a-select-option>
            </a-select>
            <!-- <j-multi-select-tag
              :disabled="formDisabled"
              v-model="model.outCompany"
              :options="companyOptions"
              placeholder="回车可搜索"
            >
            </j-multi-select-tag> -->
          </a-form-model-item>
        </a-col>
        <a-col :span="24"> <div class="title">维修工作量</div></a-col>
        <a-col :span="24">
          <a-alert type="info" showIcon message="第一个人员默认为主修人员" />
        </a-col>
      </a-row>
    </a-form-model>
    <a-button :disabled="formDisabled" v-if="!formDisabled" class="new-btn" @click="selectPersonnel" type="primary">
      <a-icon type="plus" />
      添加人员
    </a-button>
    <j-vxe-table
      ref="editableRef"
      row-number
      row-selection
      keep-source
      :height="150"
      :loading="table.loading"
      :dataSource="table.dataSource"
      :columns="table.columns"
      @valueChange="handleValueChange"
      style="margin-top: 8px;"
    >
      <template v-slot:action="props">
        <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
          <a>删除</a>
        </a-popconfirm>
      </template>
    </j-vxe-table>
    <a-col :span="24"><div class="title" style="margin-top:20px">更换备件</div></a-col>
    <a-col :span="24" v-if="!formDisabled">
      <a-form-model-item
        label="仓库"
        :labelCol="{
          xs: { span: 24 },
          sm: { span: 2 }
        }"
        :wrapperCol="{ span: 16 }"
      >
        <a-select
          style="width:50%"
          :disabled="formDisabled"
          :showSearch="true"
          optionFilterProp="label"
          v-model="storeHouse"
          placeholder="请选择仓库"
        >
          <template v-for="(item, index) of storeHouseList">
            <a-select-option :value="item.id" :label="item.storeName" :key="index">{{
              item.storeName
            }}</a-select-option>
          </template>
        </a-select>
      </a-form-model-item>
    </a-col>
    <a-button :disabled="formDisabled" v-if="!formDisabled" class="new-btn" @click="selectSparePart" type="primary">
      <a-icon type="plus" />
      选择备件
    </a-button>
    <j-vxe-table
      ref="spareRef"
      row-number
      row-selection
      keep-source
      :height="150"
      :loading="spareData.loading"
      :dataSource="spareData.dataSource"
      :columns="spareData.columns"
      style="margin-top: 8px"
    >
      <template v-slot:action="props">
        <a-popconfirm title="确定删除吗？" @confirm="handleDL(props)">
          <a>删除</a>
        </a-popconfirm>
      </template></j-vxe-table
    >

    <a-row v-if="editDisabled">
      <div class="title" style="margin-top:20px ">关联文档</div>
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
      <a-col :span="24" style="margin-top:50px">
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
            :disabled="formDisabled"
            style="width:100%"
            type="list"
            v-model="queryParam.documentType"
            dictCode="doc_type"
            placeholder="请选择文档类型"
          />
        </a-form-item>
      </a-col>
    </a-row>
    <!-- 添加人员 -->
    <j-modal
      title="添加人员"
      width="60%"
      :visible="personnelVis"
      @ok="personnelHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="personnelQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="姓名">
                <a-input v-model="personnelParam.realname"></a-input>
              </a-form-item>
            </a-col>
            <a-col :span="7">
              <a-form-item label="性别">
                <a-select v-model="personnelParam.sex" placeholder="请选择性别">
                  <a-select-option value="">请选择</a-select-option>
                  <a-select-option value="1">男</a-select-option>
                  <a-select-option value="2">女</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="4" style="float:right">
              <div style="float: right;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="personnelQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="personnelReset" icon="reload" style="margin-left: 8px">重置</a-button>
              </div>
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
        :rowSelection="{ selectedRowKeys: personnelTableSelectedRowKeys, onChange: personnelTableOnSelectChange }"
        class="j-table-force-nowrap"
        @change="personnelTableHandleTableChange"
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
      :okButtonProps="{ class: { 'jee-hidden': disableSubmit } }"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">基本信息</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model.trim="equipmentParam.condition"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="5">
              <a-form-item label="设备类型">
                <j-dict-select-tag
                  type="list"
                  v-model.trim="equipmentParam.spareType"
                  dictCode="spare_type"
                ></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="5">
              <a-form-item label="使用部门">
                <a-tree-select
                  v-model.trim="equipmentParam.field"
                  :treeData="treeData"
                  style="width: 100%"
                  :tree-data="treeData"
                  placeholder="请选择使用部门"
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
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="equipmentTable.columns"
        :dataSource="equipmentTable.dataSource"
        :pagination="equipmentTable.ipagination"
        :loading="equipmentTable.loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type: 'radio' }"
        class="j-table-force-nowrap"
        @change="handleEquipmentTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
    <!-- 选择备件 -->
    <j-modal
      title="选择备件"
      :width="900"
      :visible="spareVis"
      @ok="spareHandleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="title" style="margin-bottom:20px">选择备件</div>
      <!-- 查询区域 -->
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="spareSearchQuery">
          <a-row :gutter="24">
            <a-col :span="8">
              <a-form-item label="搜索条件">
                <a-input placeholder="编码，名称，型号" v-model="spareParam.condition"></a-input>
              </a-form-item>
            </a-col>

            <a-col :span="8">
              <a-form-item label="类型">
                <j-dict-select-tag type="list" v-model="spareParam.spareType" dictCode="spare_type"></j-dict-select-tag>
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="仓库">
                <a-select v-model="spareParam.storeHouse" allowClear
                 :showSearch="true" optionFilterProp="label"
                 >
                  <template v-for="(item, index) of storeHouseList">
                    <a-select-option :value="item.id" :label="item.storeName" :key="index">{{
                      item.storeName
                    }}</a-select-option>
                  </template>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="23">
              <span style="float: right;overflow: hidden;" class="table-page-search-submitButtons">
                <a-button type="primary" @click="spareSearchQuery" icon="search">查询</a-button>
                <a-button type="primary" @click="spareSearchReset" icon="reload" style="margin-left: 8px"
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
        :columns="spareListData.columns"
        :dataSource="spareListData.dataSource"
        :pagination="spareListData.ipagination"
        :loading="spareListData.loading"
        :rowSelection="{ selectedRowKeys: spareSelectedRowKeys, onChange: spareOnSelectChange }"
        class="j-table-force-nowrap"
        @change="spareHandleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
      </a-table>
    </j-modal>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
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
      rowMain: [],
      companyOptions: [],
      fileList: [],
      keepStatusOptions: [
        {
          label: '待维修',
          value: '0'
        },
        {
          label: '待停机维修',
          value: '1'
        },
        {
          label: '已完成',
          value: '3' //张雨航喊改成3
        }
        // {
        //   label: '维修中',
        //   value: '2'
        // },
        // {
        //   label: '外委中',
        //   value: '5'
        // },
        // {
        //   label: '待验证',
        //   value: '3'
        // },
        // {
        //   label: '已完成',
        //   value: '4'
        // }
      ],
      // 添加人员
      personnelVis: false,
      spareVis: false,
      editDisabled: false,
      personnelParam: {},
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
      table: {
        loading: false,
        dataSource: [],
        columns: [
          {
            title: '删除',
            key: 'action',
            type: JVXETypes.slot,
            fixed: 'left',
            width: '50px',
            align: 'center',
            slotName: 'action'
          },
          {
            title: '姓名',
            key: 'realname',
            width: '80px',
            type: FormTypes.normal
          },
          {
            title: '维修开始时间',
            key: 'startTime',
            width: '10%',
            type: FormTypes.datetime
          },
          {
            title: '维修完成时间',
            key: 'endTime',
            width: '10%',
            type: FormTypes.datetime
          },
          {
            title: '维修用时(小时)',
            key: 'spendTime',
            width: '10%',
            type: FormTypes.normal,
            validateRules: [{ required: false, message: '可修改单位为分钟数' }]
          },
          {
            title: '实际维修工时',
            key: 'trueSpendTime',
            width: '10%',
            type: FormTypes.input,
            placeholder: '请输入${title}',
            validateRules: [{ required: true, message: '${title}不能为空' }]
          },
          {
            title: '内容或步骤',
            key: 'content',
            width: '10%',
            type: FormTypes.input
          }
          // {
          //   title: '是否主修',
          //   key: 'main',
          //   width: '10%',
          //   type: FormTypes.checkbox
          // }
        ]
      },
      // 选择备件
      spareSelectedRowKeys: [],
      spareTableSelectionRows: [],
      storeHouseList: [],
      spareListData: {
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
        ]
      },
      // 确定备件
      spareData: {
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
            title: '备件编号',
            key: 'spareCode',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '备件名称',
            key: 'spareName',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '计量单位',
            key: 'spareCountUnit',
            width: '10%',
            type: FormTypes.normal
          },
          {
            title: '使用数量',
            key: 'useCount',
            width: '10%',
            type: FormTypes.input
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
      // 添加人员
      personnelTableSelectedRowKeys: [],
      personnelTableSelectionRows: [],
      // spareColumns: [
      //   {
      //     title: '设备编码',
      //     align: 'center',
      //     dataIndex: 'equipmentCode'
      //   },
      //   {
      //     title: '设备名称',
      //     align: 'center',
      //     dataIndex: 'equipmentName'
      //   },
      //   {
      //     title: '规格型号',
      //     align: 'center',
      //     dataIndex: 'equipmentModel'
      //   },
      //   {
      //     title: '使用部门',
      //     align: 'center',
      //     // dataIndex: 'sysOrgCode'
      //     dataIndex: 'depart'
      //   },
      //   {
      //     title: '设备类型',
      //     align: 'center',
      //     dataIndex: 'equipmentType',
      //     customRender: text => (text ? filterMultiDictText(this.dictOptions['equipmentType'], text) : '')
      //   }
      // ],
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
      queryParam: { documentType: '6' },
      equipmentParam: {},
      spareParam: {},
      storeHouse: '',
      equipmentVis: false,
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
      model: {
        stop: '0',
        fixWorkOrderWorkloads: []
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      url: {
        add: '/fixworkorder/fixWorkOrder/add',
        // edit: '/fixworkorder/fixWorkOrder/edit',
        edit: '/fixworkorder/fixWorkOrder/implement', //执行维修
        equipmentList: '/equipment/equipment/list', //设备管理分页
        queryById: '/troublereport/tabTroubleFixReport/queryById'
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
    getAction('/equipment/equipment/newTroubleList').then(res => {
      if (res.success) {
        this.dataSource = res.result.records || []
        // console.log('  this.dataSource', this.dataSource)
      }
    })
    // 往来单位
    getAction('/equipment/tabContactCompany/list', { pageNo: 1, pageSize: 99999 }).then(res => {
      // getAction('/equipment/tabContactCompany/list').then(res => {
      if (res.success) {
        let arr = [...res.result.records]
        console.log('res.供应商', res.result)
        this.companyOptions = arr
      }
    })
    // 仓库
    getAction('/stock/tabStore/list').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.storeHouseList = res.result.records || []
      }
    })
    // 这个方法是找到所有的部门信息
    queryDepartTreeList().then(res => {
      if (res.success) {
        let arr = [...res.result]
        this.treeData = arr
      }
    })
  },
  methods: {
    changeTime() {
      if (this.model.startTime && this.model.endTime) {
        let data = this.getHoursDiff(this.model.startTime, this.model.endTime)
        var h = parseInt(data)
        var m = parseInt((data - h) * 60)
        this.model.fixHour = h
        this.model.fixMin = m
      }
    },
    handleValueChange(event) {
      let row = event.row
      let _this = this
      // if (row.main == true) {
      //   _this.rowMain.push(row.main)
      // }
      // if (_this.rowMain.length >= 2) {
      //   _this.$message.warning('主修人员有且只有一个请选择一个主修人员!')
      //   row.main = false
      //   _this.rowMain = []
      // }
      let data = this.getHoursDiff(row.startTime, row.endTime)
      var h = parseInt(data)
      // var m = parseInt((data - h) * 60)
      row.spendTime = h
      // '小时' + m + '分钟'
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
      // if (isNaN(res)) throw Error('invalid dates arguments')
      return res / (1000 * 60 * 60)
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
    handleDL(props) {
      // 调用删除方法
      props.target.removeRows(props.row)
    },
    handleRemove(file) {
      const index = this.fileList.indexOf(file)
      const newFileList = this.fileList.slice()
      newFileList.splice(index, 1)
      this.fileList = newFileList
    },
    beforeUpload(file, l) {
      this.fileList = [...this.fileList, file]
      if (this.fileList.length > 0) {
        let sizeList = this.fileList.map(item => item.size)
        let sum = 0
        for (let i = 0; i < sizeList.length; i++) {
          sum += Math.round(sizeList[i] / 1024 / 1024)
        }
        this.queryParam.title = '本次上传文件总大小: ' + sum + ' MB'

        if (sum > 30) {
          this.$message.warning('文件总大小超过30MB，请修改后重新上传!')
        }
      }
      return false
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
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectionRows = selectionRows
    },
    spareOnSelectChange(spareSelectedRowKeys, spareTableSelectionRows) {
      this.spareSelectedRowKeys = spareSelectedRowKeys
      this.spareTableSelectionRows = spareTableSelectionRows
    },
    personnelTableOnSelectChange(personnelTableSelectedRowKeys, personnelTableSelectionRows) {
      this.personnelTableSelectedRowKeys = personnelTableSelectedRowKeys
      this.personnelTableSelectionRows = personnelTableSelectionRows
    },
    personnelTableHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.personnelTable.ipagination = pagination
      this.personnelLoad()
    },
    spareHandleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.spareListData.ipagination = pagination
      this.spareLoadData()
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
    selectEquipment() {
      this.equipmentLoad()
      this.equipmentVis = true
    },
    handleOk() {
      this.model.equipmentCode = this.selectionRows[0].equipmentCode
      this.model.equipmentName = this.selectionRows[0].equipmentName
      this.model.equipmentId = this.selectionRows[0].id
      this.model.equipmentType = this.selectionRows[0].equipmentType
      this.model.equipmentModel = this.selectionRows[0].equipmentModel
      // this.model.depart = this.selectionRows[0].sysOrgCode
      this.model.address = this.selectionRows[0].equipmentAddress
      console.log('this.model', this.model)
      console.log('this.selectionRows', this.selectionRows)
      this.equipmentVis = false
    },
    spareHandleOk() {
      this.spareData.dataSource = this.spareTableSelectionRows
      this.handleCancel()
    },
    personnelHandleOk() {
      var that = this
      this.personnelTableSelectionRows.forEach(item => {
        let pm = {
          workerId: item.id,
          phone: item.phone,
          username: item.username,
          realname: item.realname,
          status: item.status,
          status_dictText: item.status_dictText,
          updateBy: item.updateBy,
          fixWorkId: that.model.id ? this.model.id : ''
        }
        this.table.dataSource.push(pm)
      })
      this.personnelVis = false
    },
    handleCancel() {
      this.equipmentVis = false
      this.personnelVis = false
      this.spareVis = false
      this.queryParam = {}
      this.spareParam = {}
    },
    spareSearchQuery() {
      this.spareLoadData(1)
      // 点击查询清空列表选中行
      this.selectedRowKeys = []
      this.selectionRows = []
    },
    spareSearchReset() {
      this.spareParam = {}
      this.spareLoadData(1)
    },
    spareLoadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.spareListData.ipagination.current = 1
      }
      // var params = this.getQueryParams() //查询条件
      var param = Object.assign(this.spareParam)
      param.pageNo = this.spareListData.ipagination.current
      param.pageSize = this.spareListData.ipagination.pageSize
      this.spareListData.loading = true
      getAction('/detail/tabStockDetail/queryStockDetails', param)
        .then(res => {
          if (res.success) {
            this.spareListData.dataSource = res.result.records || res.result
            if (res.result.total) {
              this.spareListData.ipagination.total = res.result.total
            } else {
              this.spareListData.ipagination.total = 0
            }
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.spareListData.loading = false
        })
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
    personnelQuery() {
      this.personnelLoad(1)
      // 点击查询清空列表选中行
      this.personnelTableSelectedRowKeys = []
      this.personnelTableSelectionRows = []
    },
    personnelReset() {
      this.personnelParam = {}
      this.personnelLoad(1)
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
    // 关联图片
    /** upload headers */
    uploadHeaders() {
      let { originColumn: col } = this
      let headers = {}
      if (col.token === true) {
        headers['X-Access-Token'] = this.$ls.get(ACCESS_TOKEN)
      }
      return headers
    },
    /** 上传请求地址 */
    uploadAction() {
      // if (!this.originColumn.action) {
      return window._CONFIG['domianURL'] + '/sys/common/upload'
      // } else {
      //   return this.originColumn.action
      // }
    },
    handleChangeUpload(info) {
      let { originColumn: col } = this
      let { file } = info
      let value = {
        name: file.name,
        type: file.type,
        size: file.size,
        status: file.status,
        percent: file.percent
      }
      if (file.response) {
        value['responseName'] = file.response[this.responseName]
      }
      if (file.status === 'done') {
        if (typeof file.response.success === 'boolean') {
          if (file.response.success) {
            value['path'] = file.response[this.responseName]
            this.handleChangeCommon(value)
          } else {
            value['status'] = 'error'
            value['message'] = file.response.message || '未知错误'
          }
        } else {
          // 考虑到如果设置action上传路径为非jeecg-boot后台，可能不会返回 success 属性的情况，就默认为成功
          value['path'] = file.response[this.responseName]
          this.handleChangeCommon(value)
        }
      } else if (file.status === 'error') {
        value['message'] = file.response.message || '未知错误'
      }
      this.innerFile = value
    },
    // end

    uploadFilesToServer(uploadApiUrl, fileName, files) {
      let formData = new FormData()
      formData.append(fileName, files)
      //添加请求头
      const headers = {
        'Content-Type': 'multipart/form-data'
      }
      //配置头
      const request = axios.create({
        headers: headers
      })
      //开始上传
      return request
        .post(uploadApiUrl, formData)
        .then(response => {
          return Promise.resolve(response.data)
        })
        .catch(error => {
          return Promise.reject(error)
        })
    },

    selectPersonnel() {
      this.personnelLoad()
      this.personnelVis = true
    },
    personnelLoad(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.personnelTable.ipagination.current = 1
      }
      var params = {
        realname: this.personnelParam.realname,
        sex: this.personnelParam.sex,
        pageNo: this.personnelTable.ipagination.current,
        pageSize: this.personnelTable.ipagination.pageSize
      }
      this.personnelTable.loading = true
      getAction('/sys/user/list', params)
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
    selectSparePart() {
      if (this.storeHouse) {
        this.spareLoadData()
        this.spareVis = true
      } else {
        this.$message.warning('请选择仓库！')
      }
    },
    delSparePart() {
      console.log('删除')
    },
    add() {
      this.model = Object.assign({}, this.modelDefault)
      this.visible = true
    },
    edit(record) {
      this.searchEvent(record)
      this.visible = true
    },
    execute(record) {
      this.searchEvent(record)
      this.visible = true
      this.editDisabled = true
    },
    searchEvent(record) {
      getAction('/fixworkorder/fixWorkOrder/queryById?id=' + record.id).then(res => {
        if (res.success) {
          this.model = res.result
          console.log('获取编辑数据', res.result)
          this.table.dataSource = res.result.fixWorkOrderWorkloads
          this.spareData.dataSource = res.result.spareList
        }
      })
    },
    submitForm() {
      const that = this

      if (!this.model.id) {
        this.table.dataSource.forEach(item => {
          // item.workNo = item.id
          delete item.id
        })
      }
      this.table.dataSource[0].main = true
      this.model.fixWorkOrderWorkloads = this.table.dataSource
      this.model.spareList = this.spareData.dataSource
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (this.model.equipmentName) {
          this.$refs.editableRef.validateTable().then(errMap => {
            if (!errMap) {
              if (valid) {
                // 判断维修状态完成时，维修工作量是否添加人员
                if (this.table.dataSource.length == 0) {
                  this.$message.warning('请添加人员!')
                } else {
                  if (this.model.fixStatus == '3') {
                    if (!this.model.startTime) {
                      this.$message.warning('维修已完成,请选择维修开始时间!')
                    } else if (!this.model.endTime) {
                      this.$message.warning('维修已完成,请选择维修结束时间！')
                    } else {
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
                  } else {
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
                }
              }
            }
          })
        } else {
          that.$message.warning('请输入设备名称！')
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
.ant-modal-body {
  margin-bottom: 0 !important;
}
</style>
