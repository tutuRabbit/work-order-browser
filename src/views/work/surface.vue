<template>
  <div class="content">
    <div class="headers">
      <div class="headers-title"></div>
      <span class="title-out" @click="ScreenVis('on')">全屏</span>
      <span class="title-quanpin-icon" @click="ScreenVis('on')"></span>
    </div>
    <div class="content_table">
      <a-table
        :scroll="{ y: tableHeight }"
        :columns="columns"
        :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
        :data-source="listCont"
        :bordered="false"
        :pagination="false"
      >
        <template slot="planAmountSlot" slot-scope="text, record">
          <a-tooltip>
            <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left">
              <a-progress type="circle" :width="25" :percent="item.percentage" />

              <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
              <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
            </div>
          </a-tooltip>
        </template>
      </a-table>
    </div>
    <j-modal
      title="弹窗"
      width="600"
      :visible="visible"
      :fullscreen="fullscreen"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭"
    >
      <div class="headers">
        <div class="headers-title"></div>
        <span class="title-out" @click="ScreenVis('out')">退出全屏</span>
        <span class="title-quanpin-icon" @click="ScreenVis('out')"></span>
      </div>
      <div class="content_table">
        <a-table
          :scroll="{ y: tableHeight }"
          :columns="columns"
          :row-class-name="(record, index) => (index % 2 === 1 ? 'table-striped' : null)"
          :data-source="listCont"
          :bordered="false"
          :pagination="false"
        >
          <template slot="planAmountSlot" slot-scope="text, record">
            <a-tooltip>
              <div v-for="(item, index) in record.productionSchedule" :key="index" style="float:left">
                <a-progress type="circle" :width="25" :percent="item.percentage" />

                <span v-if="record.productionSchedule.length - index > 1" class="spanC">——</span>
                <p style="text-align:left;margin-left:5px">{{ item.title }}</p>
              </div>
            </a-tooltip>
          </template>
        </a-table>
      </div>
    </j-modal>
  </div>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

import { getAction } from '@/api/manage'
export default {
  name: 'surface',
  mixins: [JeecgListMixin, mixinDevice],
  components: {},
  data() {
    return {
      visible: true,
      fullscreen: true,
      tableHeight: document.body.clientHeight - 150 + 'px',
      description: '驾驶舱',
      // 表头
      dataSource: [],
      columns: [
        {
          title: '工单编号',
          align: 'center',
          dataIndex: 'code',
          width: 160,
          ellipsis: true,
          resizable: true
        },
        {
          title: '剩余数',
          align: 'center',
          dataIndex: 'planAmount',
          customRender: function(t, r, index) {
            return r.planAmount - r.offNum
          },
          ellipsis: true,
          resizable: true
        },
        {
          title: '工单状态',
          align: 'center',
          dataIndex: 'status',
          ellipsis: true,
          resizable: true,
          customRender: function(t, r, index) {
            if (r.status == 0) {
              return '未开始'
            } else if (r.status == 10) {
              return '执行中'
            } else {
              return '已结束'
            }
          }
        },
        {
          title: '产品名称',
          align: 'center',
          dataIndex: 'productName',
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品编号',
          align: 'center',
          width: 160,
          dataIndex: 'productCode',
          ellipsis: true,
          resizable: true
        },
        {
          title: '产品规格',
          align: 'center',
          dataIndex: 'productUnit',
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划数',
          align: 'center',
          dataIndex: 'planAmount',
          ellipsis: true,
          resizable: true
        },
        {
          title: '实际数',
          align: 'center',
          dataIndex: 'offNum',
          ellipsis: true,
          resizable: true
        },
        {
          title: '生产进度',
          align: 'center',
          width: 400,
          scopedSlots: { customRender: 'planAmountSlot' },
          ellipsis: true,
          resizable: true
        },
        {
          title: '计划结束时间',
          align: 'center',
          width: 180,
          dataIndex: 'planEndTime',
          ellipsis: true,
          resizable: true
        }
      ],
      url: {
        list: '/longxianda/sysCode/list',
        delete: '/longxianda/sysCode/delete',
        deleteBatch: '/longxianda/sysCode/deleteBatch',
        exportXlsUrl: '/longxianda/sysCode/exportXls',
        importExcelUrl: 'longxianda/sysCode/importExcel'
      },
      listCont: []
    }
  },
  created() {
    this.init()
  },
  computed: {},
  methods: {
    clickT() {
      this.visible = true
      console.log('this.visible', this.visible)
    },
    handleOk() {},
    handleCancel() {
      this.visible = false
    },
    ScreenVis(type) {
      if (type == 'on') {
        this.visible = true
      } else {
        this.visible = false
      }
    },
    initDictConfig() {},
    init() {
      var that = this
      let httpurl = '/longxianda/sysWorkOrder/list'
      let pm = {
        order: 'desc',
        column: 'createTime',
        pageNo: 1,
        pageSize: 100000
      }
      getAction(httpurl, pm)
        .then(res => {
          console.log(res.result.records)
          that.listCont = res.result.records
        })
        .finally(() => {})
    }
  }
}
</script>
<style lang="less" scoped>
@import '~@assets/less/common.less';
::v-deep {
  .ant-modal-body {
    padding: 0;
    height: 100% !important;
  }
  .ant-modal-content {
    background-image: url(img/gd_background.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
  }
  .ant-modal-content > .ant-modal-header {
    display: none;
  }
  .ant-modal-content > .ant-modal-footer {
    display: none;
  }
  .ant-modal-close {
    display: none;
  }
}
::v-deep .ant-table-thead > tr > th,
.ant-table-tbody > tr > td {
  padding: 8px 8px;
  overflow-wrap: break-word;
  color: #f5f6ff;
  background: #3f456b;
  border: none;
}
::v-deep .ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: #3355ff33;
  color: #383838;
}
::v-deep .ant-progress-circle .ant-progress-text {
  color: #fff;
  font-size: 12px;
}
::v-deep .ant-table-tbody .ant-table-row td {
  padding-top: 8px;
  padding-bottom: 8px;
  color: #f5f6ff;
  background: #1d2753;
  box-shadow: 0px -1px 0px 0px rgba(51, 85, 255, 0.2);
}
::v-deep .ant-table-tbody > tr > td {
  border: none;
}
.ant-progress-circle .ant-progress-text {
  color: #ffffffa6;
}
.ant-table-striped ::v-deep(.table-striped) td {
  background-color: #17700efa;
}
.content {
  height: 100vh;
  width: 100%;
  background-image: url(img/gd_background.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
}
.headers {
  position: relative;
  background-repeat: no-repeat;
  background-image: url(img/bg_Title.png);
  background-size: 100% 100%;
  height: 8vh;
  padding-top: 1.5vh;
  font-size: 18px;
  color: rgb(75, 239, 255);
  font-weight: 600;
  .header-bg {
    font-size: 18px;
    float: right;
    color: rgb(75, 239, 255);
    font-weight: 600;
  }
  .headers-title {
    margin-left: 42%;
    background-image: url(img/车间生产监控大屏.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    height: 46px;
    width: 296px;
    padding-top: 1.5vh;
    text-align: center;
    font-size: 14px;
    font-family: Source Han Sans CN;
    font-weight: 400;
    color: #6bb0ff;
  }
  .title-out {
    position: absolute;
    top: 3vh;
    right: 6vh;
    font-size: 14px;
    font-family: Source Han Sans CN;
    color: #6bb0ff;
    &:hover {
      cursor: pointer;
    }
  }
  .title-out-icon {
    position: absolute;
    top: 1.5vh;
    right: 6vh;
    width: 14px;
    height: 14px;
    background-image: url(img/out.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    &:hover {
      cursor: pointer;
    }
  }
  .title-quanpin-icon {
    position: absolute;
    top: 3.5vh;
    right: 3.5vh;
    width: 14px;
    height: 14px;
    background-image: url(img/quanpin.png);
    background-size: 100% 100%;
    background-repeat: no-repeat;
    &:hover {
      cursor: pointer;
    }
  }
}

.content_table {
  // overflow-y: auto;
  padding: 10px;
  background-image: url(img/bg_border.png);
  background-size: 100% 100%;
  background-repeat: no-repeat;
  height: 88vh;
  margin: 2vh 2vh 2vh 2vh;
}
.ant-table-cell {
  background: rgb(8, 113, 132);
  opacity: 0.6;
}
.ant-table-cell- {
  background: rgb(5, 17, 46);
  opacity: 0.6;
  color: #fff;
}
.ant-table-thead > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr.ant-table-row-hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-thead > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td,
.ant-table-tbody > tr:hover:not(.ant-table-expanded-row):not(.ant-table-row-selected) > td {
  background: #e6f7ff;
  opacity: 0.6;
  color: #333333;
}
</style>
