<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
      </a-table>
    </div>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'

export default {
  name: 'operatingRecordList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '点检明细',
      // 表头
      columns: [
        {
          title: '名称',
          align: 'center',
          dataIndex: 'keepPart'
        },
        {
          title: '记录方式',
          align: 'center',
          dataIndex: 'ad'
        },
        {
          title: '参考值',
          align: 'center',
          dataIndex: 's'
        },
        {
          title: '上限',
          align: 'center',
          dataIndex: 'dd'
        },
        {
          title: '下限',
          align: 'center',
          dataIndex: 'keepStandard'
        }
      ],
      url: {
        list: '/spot/tabSpotCheckItems/queryById?id='
      }
    }
  },
  created() {},
  methods: {
    details(val) {
      console.log('val', val)
      if (val) {
        getAction(this.url.list + val.id).then(res => {
          if (res.result) {
            this.dataSource = res.result
          } else {
            this.dataSource = null
          }
        })
      }
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
