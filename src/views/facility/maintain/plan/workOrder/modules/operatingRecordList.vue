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
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
      </a-table>
      <!-- 
        :pagination="ipagination"
         -->
    </div>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
import { mixinDevice } from '@/utils/mixin'

export default {
  name: 'operatingRecordList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {},
  data() {
    return {
      description: '操作记录',
      // 表头
      columns: [
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
      ],
      url: {
        list: '/realKeepOrder/tabKeepOrderReal/queryById?id='
      },
      likeId: ''
    }
  },
  created() {},
  methods: {
    details(val) {
      if (val) {
        this.likeId = val.id
      }
      if (this.likeId) {
        getAction(this.url.list + this.likeId).then(res => {
          if (res.result.partRelationList) {
            this.dataSource = res.result.partRelationList
          } else {
            this.dataSource = null
          }
        })
      }
    },
    handleTableChange(pagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      //TODO 筛选
      console.log(pagination)
      if (Object.keys(sorter).length > 0) {
        this.isorter.column = sorter.field
        this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = pagination
      this.loadData()
    },
    loadData(arg) {
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      this.loading = true
      getAction(this.url.list + this.likeId)
        .then(res => {
          if (res.success) {
            this.dataSource = res.result.partRelationList
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
