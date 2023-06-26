<template>
  <a-card :bordered="false" style="margin-left:-20px">
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
      >
        <!-- 
        :loading="loading"
        :pagination="ipagination"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
         -->
      </a-table>
    </div>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'

export default {
  name: 'inventoryList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '仓库',
      // 表头
      columns: [
        {
          title: '仓库',
          align: 'center',
          dataIndex: 'storeHouse'
        },
        {
          title: '库存数量',
          dataIndex: 'stockCount',
          align: 'center'
        },
        {
          title: '位置编码',
          dataIndex: 'type',
          align: 'center'
        }
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
        queryStockById: '/info/tabStock/queryStockById?id=' //备件台账-下方明细的库存
      }
    }
  },
  created() {},
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    details(val) {
      console.log('val', val)
      // 出入库明细
      getAction(this.url.queryStockById + val.id).then(res => {
        if (res.success) {
          this.dataSource = res.result
        } else {
          this.$message.warning(res.message)
        }
      })
    }
  }
}
</script>
<style scoped>
.ant-card {
  margin-left: -30px;
  margin-right: -30px;
}
.ant-card-body {
  padding-top: 0;
}
</style>
