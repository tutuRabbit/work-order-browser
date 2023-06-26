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
        :pagination="ipagination"
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
      description: '操作记录',
      // 表头
      columns: [
        {
          title: '设备巡检进度',
          align: 'center',
          dataIndex: 'keepPart'
        },
        {
          title: '计划时间',
          align: 'center',
          dataIndex: 'keepStandard'
        },
        {
          title: '开始时间',
          align: 'center',
          dataIndex: 'strateTime'
        },
        {
          title: '结束时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '巡检人',
          align: 'center',
          dataIndex: 'person'
        },
        {
          title: '是否跳过',
          align: 'center',
          dataIndex: 'true'
        }
      ],
      url: {
        list: '/realKeepOrder/tabKeepOrderReal/queryById?id='
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
        getAction(this.url.list + val.id).then(res => {
          if (res.result.partRelationList) {
            this.dataSource = res.result.partRelationList
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
