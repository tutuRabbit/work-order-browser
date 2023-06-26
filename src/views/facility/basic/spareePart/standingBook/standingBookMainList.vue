<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper" style="margin-bottom:10px">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="3" :lg="5" :md="6" :sm="22">
            <a-select v-model="queryParam.outOrIn" @change="loadData" allowClear>
              <a-select-option value="0">全部</a-select-option>
              <a-select-option value="1">入库</a-select-option>
              <a-select-option value="2">出库</a-select-option>
            </a-select>
          </a-col>
          <a-col :xl="3" :lg="5" :md="6" :sm="22">
            <a-select v-model="queryParam.storeHouse" placeholder="选择仓库" @change="loadData" allowClear>
              <template v-for="(item, index) of storeHouseList">
                <a-select-option :value="item.id" :key="index">{{ item.storeName }}</a-select-option>
              </template>
            </a-select>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :bordered="false"
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <!-- <span slot="action" slot-scope="text, record">
          <a @click="handleDetail(record)">详情</a>
        </span> -->
      </a-table>
    </div>
    <!-- table区域-end -->
    <!-- 出入库详情 -->
    <!-- <detail-Form ref="detailForm"></detail-Form> -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
// import { detailForm } from './TabStockModal'
import { mixinDevice } from '@/utils/mixin'

export default {
  name: 'standingBookMainList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    // detailForm
    // JeecgOrderDMainList,
    // JeecgOrderCustomerModal
  },
  data() {
    return {
      description: '出入库明细',
      storeHouseList: [],
      // 表头
      columns: [
        // {
        //   title: '详情',
        //   dataIndex: 'action',
        //   align: 'center',
        //   fixed: 'left',
        //   key: 'id',
        //   width: 147,
        //   scopedSlots: { customRender: 'action' }
        // },
        {
          title: '出入库单号',
          align: 'center',
          width: 100,
          dataIndex: 'orderNum'
        },
        {
          title: '仓库',
          align: 'center',
          dataIndex: 'storeHouse'
        },
        {
          title: '方向',
          dataIndex: 'direction',
          align: 'center'
        },
        {
          title: '类型',
          dataIndex: 'type',
          align: 'center'
        },
        {
          title: '入库',
          dataIndex: 'inCount',
          align: 'center'
        },
        {
          title: '出库',
          dataIndex: 'outCount',
          align: 'center'
        },
        {
          title: '库存',
          dataIndex: 'stockCount',
          align: 'center'
        },
        {
          title: '单价',
          dataIndex: 'unitPrice',
          align: 'center'
        },
        {
          title: '金额',
          dataIndex: 'amount',
          align: 'center'
        },
        {
          title: '经办人',
          dataIndex: 'operator',
          align: 'center'
        }
        // {
        //   title: '关联单号',
        //   dataIndex: 'telphone',
        //   align: 'center'
        // }
      ],
      loading: false,
      dataSource: [],
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
      url: {
        list: '/spare/tabSpare/queryTabSpareExportVoById',
        delete: '/test/order/deleteCustomer',
        OutInDetail: '/info/tabStock/OutInDetail', //备件台账-出入库明细
        deleteBatch: '/test/order/deleteBatchCustomer'
      },
      spareId: '',
      queryParam: {
        outOrIn: '0'
      }
    }
  },
  created() {
    // 仓库
    getAction('/stock/tabStore/list').then(res => {
      console.log('getAction', res)
      if (res.success) {
        this.storeHouseList = res.result.records || []
      }
    })
  },
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    handleDetail(row) {
      // console.log('row', row)
      // this.$refs.detailForm.edit(row)
      // this.$refs.detailForm.title = '详情'
      // this.$refs.detailForm.disableSubmit = true
    },
    details(val) {
      console.log('val', val)
      this.spareId = val.id
      let pm = {
        outOrIn: '0',
        pageNo: 1,
        pageSize: 999,
        spareId: val.id
      }
      // 出入库明细
      getAction(this.url.OutInDetail, pm).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
        }
      })
    },
    loadData(arg) {
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = Object.assign(this.queryParam)
      params.spareId = this.spareId
      params.pageNo = this.ipagination.current
      params.pageSize = this.ipagination.pageSize
      console.log('params', params)
      getAction(this.url.OutInDetail, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        } else {
          this.dataSource = null
        }
      })
    }
  }
}
</script>
<style scoped>
.ant-card {
  /* margin-left: -30px;
  margin-right: -30px; */
}
.ant-card-body {
  padding-top: 0;
}
</style>
