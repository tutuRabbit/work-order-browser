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
  name: 'checkRecordList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '点检记录',
      // 表头
      columns: [
        {
          title: '点检时间',
          align: 'center',
          dataIndex: 'spotCheckTime'
        },
        {
          title: '点检人',
          align: 'center',
          dataIndex: 'spotCheckPerson'
        },
        {
          title: '备注',
          align: 'center',
          maxWidth: 500,
          ellipsis: true,
          dataIndex: 'remark'
        }
        // {
        //   title: '总数',
        //   align: 'center',
        //   dataIndex: 's'
        // },
        // {
        //   title: '漏检',
        //   align: 'center',
        //   dataIndex: 'dd'
        // },
        // {
        //   title: '异常',
        //   align: 'center',
        //   dataIndex: 'keepStandard'
        // }
      ],
      url: {
        list: '/spot_record/tabSpotCheckRecords/list',
        queryById: '/spot_record/tabSpotCheckRecords/queryById'
      }
    }
  },
  created() {},
  beforeCreate() {
    this.disableMixinCreated = true
  },
  methods: {
    details(val) {
      if (val) {
        getAction(this.url.list, { spotCheckItemId: val.id }).then(res => {
          if (res.success) {
            this.dataSource = res.result.records
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
