<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="出库单号">
              <a-input :allowClear="true" placeholder="请输入" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="出库类型">
              <j-dict-select-tag
                :allowClear="true"
                v-model="queryParam.libraryType"
                placeholder="请选择"
                dictCode="out_stock_type"
              />
              <!-- <a-input placeholder="请输入" v-model="queryParam.libraryType"></a-input> -->
            </a-form-item>
          </a-col>
          <a-col :xl="8" :lg="9" :md="10" :sm="24">
            <a-form-item label="出库时间">
              <a-range-picker
                :allowClear="true"
                v-model="queryTime"
                style="width:100%"
                :show-time="{ format: 'HH:mm' }"
                @change="onChangeRangeDate"
                format="YYYY-MM-DD HH:mm:ss"
              />
              <!-- <a-input placeholder="请输入" v-model="queryParam.libraryTime"></a-input> -->
            </a-form-item>
          </a-col>
          <!-- <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="备注">
                <a-input placeholder="请输入" v-model="queryParam.remarks"></a-input>
              </a-form-item>
            </a-col>
          </template> -->
          <a-col :xl="4" :lg="5" :md="6" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!-- <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a> -->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus" v-has="'SysLibraryOutList：add'">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('出库表')" v-has="'SysLibraryOutList：export'"
        >导出</a-button
      >
      <a-button type="primary" icon="import" @click="importVisible = true" v-has="'SysLibraryOutList：import'"
        >导入</a-button
      > -->
      <!-- <a-upload
        v-has="'SysLibraryOutList：import'"
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0" v-has="'SysLibraryOutList：delete'">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete" />
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
          <a-divider type="vertical" />
          <a-popover title="自定义列" trigger="click" placement="leftBottom">
            <template slot="content">
              <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
                <a-row style="width: 400px">
                  <template v-for="(item, index) in defColumns">
                    <template v-if="item.key != 'rowIndex' && item.dataIndex != 'action'">
                      <a-col :span="12" :key="index"
                        ><a-checkbox :value="item.dataIndex"
                          ><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox
                      ></a-col>
                    </template>
                  </template>
                </a-row>
              </a-checkbox-group>
            </template>
            <a><a-icon type="setting" />设置</a>
          </a-popover>
        </span>
      </div>
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :components="components"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <div slot="filterDropdown">
          <a-card>
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row style="width: 400px">
                <template v-for="(item, index) in defColumns">
                  <template v-if="item.key != 'rowIndex' && item.dataIndex != 'action'">
                    <a-col :span="12" :key="index"
                      ><a-checkbox :value="item.dataIndex"
                        ><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox
                    ></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type="setting" :style="{ fontSize: '16px', color: '#108ee9' }" />
        <span slot="action" slot-scope="text, record">
          <a @click="handlePrint(record)">打印</a>
          <a-divider type="vertical" />
          <a @click="handleEdit(record)" v-has="'SysLibraryOutList：edit'">编辑</a>
          <a-divider type="vertical" v-has="'SysLibraryOutList：edit'" />
          <a-popconfirm
            title="确定删除吗?"
            @confirm="() => handleDelete(record.id)"
            v-has="'SysLibraryOutList：delete'"
          >
            <a>删除</a>
          </a-popconfirm>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)" v-has="'SysLibraryOutList：delete'">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 导入 -->
    <j-modal title="导入出库" width="45%" :visible="importVisible" cancelText="关闭" @cancel="handleCancel">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :span="24">
              <p>操作步骤:</p>
              <p>1.下载<a @click="downFiles">《导入出库模板》</a></p>
            </a-col>
            <a-col :span="24">
              <p>2.打开下载表，将对应信息填入或粘贴进本表。为保证导入成功，请使用纯文本或数字。</p>
            </a-col>
            <a-col :span="24">
              <p>3.信息输入完毕并保存后，点击下方【上传文件】按钮选择已保存的excel文档。</p>
            </a-col>
            <a-col :span="24">
              <p>4. 点击【导入】</p>
            </a-col>
            <a-col :span="4">
              <a-upload
                name="file"
                :multiple="false"
                :fileList="fileList"
                :remove="handleRemove"
                :beforeUpload="beforeUpload"
              >
                <a-button type="info" icon="upload" :disabled="fileList.length != 0 ? true : false">选择文件</a-button>
              </a-upload>
            </a-col>
            <a-col :span="20">附件支持类型：XLSX，最大不能超过10M</a-col>
          </a-row>
        </a-form>
      </div>
      <template slot="footer">
        <a-button @click="handleCancel">关闭</a-button>
        <a-button @click="submitImport" :disabled="fileList.length != 0 ? false : true" type="primary">导入</a-button>
      </template>
    </j-modal>
    <!-- 表单区域 -->
    <sysLibrary-modal ref="modalForm" @ok="modalFormOk"></sysLibrary-modal>
    <a-modal
      title="出库单打印"
      width="80%"
      :bodyStyle="bodyStyle"
      :visible="printVis"
      @cancel="printVis = false"
      :footer="null"
    >
      <split-pane
        :min-percent="20"
        :default-percent="50"
        split="vertical"
        style="font-family: Microsoft YaHei;
          color: #000; "
      >
        <template slot="paneL">
          <div class="pane" style="padding-top:20px">
            <div class="no-print" style="text-align: right">
              <a-button v-print="'#inspectPrint'" ghost type="primary">打印</a-button>
            </div>
            <h3 style="color:red">套打示例</h3>
            <section>
              <div style="text-align: center">
                <p style="font-size: 24px;">重庆川仪十七厂外购/外协件送检单</p>
              </div>
              <a-col :md="24" :sm="24">
                <div class="sign" style="text-align: left;height: inherit">
                  <a-col :span="12">
                    <span>
                      供方单位:
                    </span>
                    <span>
                      {{ this.model.supplierUnit }}
                    </span>
                  </a-col>
                  <a-col :span="12" style="text-align:right">
                    <span>
                      送检日期:
                    </span>
                    <span> {{ this.model.libraryTime }}</span>
                  </a-col>
                  <a-col :span="24" style="margin-top: 20px">
                    <table class="table">
                      <thead class="ant-table-thead" border="1">
                        <tr>
                          <th class="ant-table-cell">材料/零件名称</th>
                          <th class="ant-table-cell">规格、型号、图号</th>
                          <th class="ant-table-cell">材质</th>
                          <th class="ant-table-cell">计划号</th>
                          <th class="ant-table-cell">合同数量</th>
                          <th class="ant-table-cell">送检数量</th>
                          <th class="ant-table-cell">合格数量</th>
                          <th class="ant-table-cell">不合格数量</th>
                        </tr>
                      </thead>
                      <tbody class="ant-table-tbody" v-if="printDataSource.length == 0">
                        <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                          <td colspan="100" class="ant-table-cell">
                            <div class="ant-empty ant-empty-normal">
                              <div class="ant-empty-image"></div>
                              <div class="ant-empty-description">暂无数据</div>
                            </div>
                          </td>
                        </tr>
                      </tbody>
                      <tbody class="ant-table-tbody" v-else>
                        <tr v-for="(item, index) in printDataSource" :key="index">
                          <td class="ant-table-cell">{{ item.productName }}</td>
                          <td class="ant-table-cell">{{ item.productNorms }}</td>
                          <td class="ant-table-cell">{{ item.material }}</td>
                          <td class="ant-table-cell">{{ item.planNumber }}</td>
                          <td class="ant-table-cell">{{ item.libraryNumber }}</td>
                          <td class="ant-table-cell">{{ item.libraryNumber }}</td>
                          <td class="ant-table-cell">{{ item.remarks }}</td>
                          <td class="ant-table-cell">{{ item.remarks }}</td>
                        </tr>
                        <tr>
                          <td class="ant-table-cell">备注</td>
                          <td colspan="7" class="ant-table-cell"></td>
                        </tr>
                        <tr>
                          <td colspan="1" style="border-style:none">
                            <span>供方经办人:</span>
                            <span> {{ this.model.deliveryPerson }}</span>
                          </td>
                          <td colspan="2" style="border-style: none">
                            <span>采购员:</span>
                            <span> {{ this.model.purchaser }}</span>
                          </td>
                          <td colspan="2" style="border-style: none">
                            <span>库房:</span>
                          </td>
                          <td colspan="2" style="border-style: none">
                            <span>生产科计划员:</span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </a-col>
                </div>
              </a-col>
            </section>
            <section ref="print" id="inspectPrint">
              <a-col :md="24" :sm="24">
                <div class="sign" style="text-align: left;height: inherit">
                  <a-col :span="12">
                    <span style="padding-left:100px">
                      {{ this.model.supplierUnit }}
                    </span>
                  </a-col>
                  <a-col :span="24" style="margin-top: 20px">
                    <table class="table">
                      <thead class="ant-table-thead" style="display:none;border:none">
                        <tr>
                          <th class="ant-table-cell">材料/零件名称</th>
                          <th class="ant-table-cell">规格、型号、图号</th>
                          <th class="ant-table-cell">材质</th>
                          <th class="ant-table-cell">计划号</th>
                          <th class="ant-table-cell">合同数量</th>
                          <th class="ant-table-cell">送检数量</th>
                          <th class="ant-table-cell">合格数量</th>
                          <th class="ant-table-cell">不合格数量</th>
                        </tr>
                      </thead>
                      <tbody class="ant-table-tbody" v-if="printDataSource.length == 0">
                        <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                          <td colspan="100" class="ant-table-cell">
                            <div class="ant-empty ant-empty-normal">
                              <div class="ant-empty-image"></div>
                              <div class="ant-empty-description">暂无数据</div>
                            </div>
                          </td>
                        </tr>
                      </tbody>
                      <tbody class="ant-table-tbody" v-else style="border:none">
                        <tr v-for="(item, index) in printDataSource" :key="index">
                          <td class="ant-table-cell" style="border:none">{{ item.productName }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.productNorms }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.material }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.planNumber }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.remarks }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.remarks }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.remarks }}</td>
                          <td class="ant-table-cell" style="border:none">{{ item.remarks }}</td>
                        </tr>
                        <tr>
                          <td style="border-style:none">
                            <span style="padding-left:80px">{{ this.model.deliveryPerson }}</span>
                          </td>
                          <td style="border-style: none">
                            <span style="padding-left:10px"> {{ this.model.purchaser }}</span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </a-col>
                </div>
              </a-col>
            </section>
          </div>
        </template>
        <template slot="paneR">
          <div class="pane" style="padding-top:20px">
            <div class="no-print" style="text-align: right">
              <a-button v-print="'#printContent'" ghost type="primary">打印</a-button>
            </div>
            <section ref="print" id="printContent" class="abcdefg">
              <div style="text-align: center">
                <p style="font-size: 24px;">重庆川仪流量仪表送货单</p>
              </div>
              <!--签字-->
              <a-col :md="24" :sm="24">
                <div class="sign" style="text-align: left;height: inherit">
                  <a-col :span="24">
                    <span>
                      送货单位:
                    </span>
                    <span>
                      {{ this.model.supplierUnit }}
                    </span>
                  </a-col>
                  <a-col :span="24" style="margin-top: 20px">
                    <span>
                      送货日期:
                    </span>
                    <span> {{ this.model.libraryTime }}</span>
                  </a-col>
                  <a-col :span="24" style="margin-top: 20px">
                    <table class="table">
                      <thead class="ant-table-thead" border="1">
                        <tr>
                          <th class="ant-table-cell">序号</th>
                          <th class="ant-table-cell">单据编号</th>
                          <th class="ant-table-cell">物料名称</th>
                          <th class="ant-table-cell">规格型号</th>
                          <th class="ant-table-cell">数量</th>
                          <th class="ant-table-cell">备注</th>
                        </tr>
                      </thead>
                      <tbody class="ant-table-tbody" v-if="printDataSource.length == 0">
                        <tr class="ant-table-placeholder" draggable="true" style="cursor: move;">
                          <td colspan="100" class="ant-table-cell">
                            <div class="ant-empty ant-empty-normal">
                              <div class="ant-empty-image"></div>
                              <div class="ant-empty-description">暂无数据</div>
                            </div>
                          </td>
                        </tr>
                      </tbody>
                      <tbody class="ant-table-tbody" v-else>
                        <tr v-for="(item, index) in printDataSource" :key="index">
                          <td class="ant-table-cell">{{ index + 1 }}</td>
                          <td class="ant-table-cell">{{ item.planNumber }}</td>
                          <td class="ant-table-cell">{{ item.productName }}</td>
                          <td class="ant-table-cell">{{ item.productNorms }}</td>
                          <td class="ant-table-cell">{{ item.libraryNumber }}</td>
                          <td class="ant-table-cell">{{ item.remarks }}</td>
                        </tr>
                        <tr v-for="count in 6" :key="count">
                          <td class="ant-table-cell"></td>
                          <td class="ant-table-cell"></td>
                          <td class="ant-table-cell"></td>
                          <td class="ant-table-cell"></td>
                          <td class="ant-table-cell"></td>
                          <td class="ant-table-cell"></td>
                        </tr>
                        <tr>
                          <td colspan="3" class="ant-table-thead" style="border-right-style: none">
                            <span>送货人:</span>
                            <span> {{ this.model.deliveryPerson }}</span>
                          </td>
                          <td colspan="3" class="ant-table-thead" style="border-left-style: none">
                            <span>接收人:</span>
                            <span></span>
                          </td>
                        </tr>
                        <tr>
                          <td colspan="3" style="border-style:none">
                            <span>日期:</span>
                            <span></span>
                          </td>
                          <td colspan="3" style="border-style: none">
                            <span>日期:</span>
                            <span></span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </a-col>
                </div>
              </a-col>
            </section>
          </div>
        </template>
      </split-pane>
    </a-modal>
  </a-card>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable'
import Vue from 'vue'
import '@/assets/less/TableExpand.less'
import splitPane from 'vue-splitpane'
import SysLibraryModal from './modules/SysLibraryOutModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction, uploadAction, downFile } from '@/api/manage'

export default {
  name: 'SysLibraryOutList',
  mixins: [JeecgListMixin],
  components: { splitPane, SysLibraryModal, VueDraggableResizable },
  data() {
    this.components = {
      header: {
        cell: (h, props, children) => {
          const { key, ...restProps } = props
          const col = this.columns.find(col => {
            const k = col.dataIndex || col.key
            return k === key
          })

          if (!col || !col.width) {
            return h('th', { ...restProps }, [...children])
          }

          const dragProps = {
            key: col.dataIndex || col.key,
            class: 'table-draggable-handle',
            attrs: {
              w: 10,
              x: col.width, //定义元素的初始x位置。
              z: 1,
              axis: 'x',
              draggable: true,
              resizable: false
            },
            on: {
              dragging: (x, y) => {
                col.width = Math.max(x, 1)
              }
            }
          }
          const drag = h('vue-draggable-resizable', { ...dragProps })
          console.log('drag', drag)
          return h('th', { ...restProps, class: 'resize-table-th' }, [...children, drag])
        }
      }
    }
    return {
      printColumns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '单据编号',
          align: 'center',
          dataIndex: 'planNumber'
        },
        {
          title: '物料名称',
          align: 'center',
          dataIndex: 'productName'
        },
        {
          title: '规格型号',
          align: 'center',
          dataIndex: 'productNorms'
        },
        {
          title: '数量',
          align: 'center',
          dataIndex: 'libraryNumber'
        },
        {
          title: '备注',
          align: 'center',
          dataIndex: 'remarks'
        }
      ],
      printDataSource: [],
      model: {},
      fileList: [],
      printVis: false,
      bodyStyle: {
        padding: '0',
        //update-begin---author:wangshuai ---date:20220104  for：[JTC-411]火狐 分屏 图片大时，与按钮重叠，样式不好------------
        // height: window.innerHeight - 240 + 'px'
        height: window.innerHeight + 'px'
        //update-begin---author:wangshuai ---date:20220104  for：[JTC-411]火狐 分屏 图片大时，与按钮重叠，样式不好------------
      },
      importVisible: false,
      queryTime: [],
      description: '出库表管理页面',
      // 表头
      columns: [],
      //列设置
      settingColumns: [],
      defColumns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '出库单号',
          align: 'center',
          dataIndex: 'code',
          width: 180,
          ellipsis: true
        },
        {
          title: '出库类型',
          align: 'center',
          dataIndex: 'libraryTypeName',
          width: 180,
          ellipsis: true
        },
        {
          title: '出库时间',
          align: 'center',
          width: 180,
          dataIndex: 'libraryTime',
          ellipsis: true
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remarks',
          ellipsis: true
        },
        {
          title: '创建时间',
          align: 'center',
          width: 180,
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'createTime'
        },
        {
          title: '创建人',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'userName'
        },
        // {
        //   title: '最后更新时间',
        //   align: 'center',
        //   maxWidth: 500,
        //   ellipsis: true,
        //   dataIndex: 'updateTime'
        // },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'action'
          }
        }
      ],
      url: {
        list: '/longxianda/sysLibrary/list?type=2',
        delete: '/longxianda/sysLibrary/delete',
        deleteBatch: '/longxianda/sysLibrary/deleteBatch',
        exportXlsUrl: '/longxianda/sysLibrary/exportXls',
        importExcelUrl: 'longxianda/sysLibrary/importExcel',
        download: '/attachment/tabAttachment/download?id=' //出库导入模板
      },
      superFieldList: []
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  created() {
    this.getSuperFieldList()
    this.initColumns()
    this.departName()
  },
  methods: {
    departName() {
      let url = '/sys/user/getCurrentUserDeparts'
      getAction(url).then(res => {
        if (res.success) {
          let parentName = res.result.parentName
          localStorage.setItem('parentName', parentName)
          console.log('localStorage.getItem("parentName")', localStorage.getItem('parentName'))
        }
      })
    },
    //列设置更改事件
    onColSettingsChange(checkedValues) {
      var key = this.$route.name + ':colsettings'
      Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
      this.settingColumns = checkedValues
      const cols = this.defColumns.filter(item => {
        if (item.key == 'rowIndex' || item.dataIndex == 'action') {
          return true
        }
        if (this.settingColumns.includes(item.dataIndex)) {
          return true
        }
        return false
      })
      this.columns = cols
    },
    initColumns() {
      //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
      var key = this.$route.name + ':colsettings'
      let colSettings = Vue.ls.get(key)
      if (colSettings == null || colSettings == undefined) {
        let allSettingColumns = []
        this.defColumns.forEach(function(item, i, array) {
          allSettingColumns.push(item.dataIndex)
        })
        this.settingColumns = allSettingColumns
        this.columns = this.defColumns
      } else {
        this.settingColumns = colSettings
        const cols = this.defColumns.filter(item => {
          if (item.key == 'rowIndex' || item.dataIndex == 'action') {
            return true
          }
          if (colSettings.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.columns = cols
      }
    },
    handlePrint(record) {
      console.log('record', record)
      var that = this
      let httpurl = '/longxianda/sysLibrary/queryById?id=' + record.id
      getAction(httpurl)
        .then(res => {
          if (res.code == 200) {
            that.model = res.result
            console.log('  that.model', that.model)
            that.printDataSource = that.model.libraryDetaileds
          } else {
            that.$message.warning(res.message)
          }
        })
        .finally(() => {
          that.confirmLoading = false
        })
      this.printVis = true
    },
    handleCancel() {
      this.importVisible = false
      this.fileList = []
    },
    downFiles() {
      downFile(this.url.download).then(data => {
        if (!data) {
          this.$message.warning('文件下载失败')
          return
        }
        if (typeof window.navigator.msSaveBlob !== 'undefined') {
          window.navigator.msSaveBlob(new Blob([data], { type: 'application/vnd.ms-excel' }), fileName + '.xls')
        } else {
          let url = window.URL.createObjectURL(new Blob([data], { type: 'application/vnd.ms-excel' }))
          let link = document.createElement('a')
          link.style.display = 'none'
          link.href = url
          link.setAttribute('download', '出库导入模板' + Math.round(new Date().getTime()).toString())
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link) //下载完成移除元素
          window.URL.revokeObjectURL(url) //释放掉blob对象
        }
      })
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
        if (sum > 10) {
          this.$message.warning('文件总大小超过10MB，请修改后重新上传!')
        }
      }
      return false
    },
    submitImport() {
      let pm = new FormData()
      if (this.fileList.length != 0) {
        let { fileList } = this
        fileList.forEach(file => {
          pm.append('file', file)
        })
      }
      uploadAction(this.url.importExcelUrl, pm).then(res => {
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
      })
      this.importVisible = false
      this.fileList = []
    },
    onChangeRangeDate(value, dateString) {
      this.queryTime = value
      this.queryParam.startTime = dateString[0]
      this.queryParam.endTime = dateString[1]
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'code', text: '出库单号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'libraryType', text: '出库类型', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'libraryTime', text: '出库时间' })
      fieldList.push({ type: 'string', value: 'remarks', text: '备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
.split-pane {
  padding: 20px;
}
.pane {
  padding: 20px;
}

.table {
  table-layout: auto;
  width: 100%;
}
.ant-table-cell {
  border: 1px solid #ddd;
}
.ant-table-thead > tr > th,
.ant-table-tbody > tr > td {
  padding: 10px;
}
.ant-table-thead > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-thead > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: #ffffff;
  opacity: 0.6;
  color: #333333;
}
</style>

<style>
.table-draggable-handle {
  height: 100% !important;
  left: auto !important;
  right: -5px;
  cursor: col-resize;
  touch-action: none;
  border: none;
  position: absolute;
  transform: none !important;
  bottom: 0;
}
.resize-table-th {
  position: relative;
}
</style>
