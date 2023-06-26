<template>
  <a-card :bordered="false" style="margin-left:-20px">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper" style="margin-bottom:10px">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24"> </a-row>
      </a-form>
    </div>
    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :bordered="false"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
      >
        <!-- 
        :pagination="ipagination"
        @change="handleTableChange"
        rowKey="id"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        :loading="loading"
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
  name: 'operatingRecordList',
  mixins: [JeecgListMixin],
  components: {},
  data() {
    return {
      description: '操作记录',
      // 表头
      columns: [
        {
          title: '处理时间',
          align: 'center',
          width: 200,
          dataIndex: 'createTime'
        },
        {
          title: '操作用户',
          align: 'center',
          dataIndex: 'createBy'
          // dataIndex: 'userId'
        },
        {
          title: '操作',
          align: 'center',
          dataIndex: 'operation'
        },
        {
          title: '操作结果',
          align: 'center',
          dataIndex: 'operationResult'
        }
      ],
      url: {
        list: '/troublereport/troubleHistory/list'
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
      getAction(this.url.list, {
        troubleId: val.id,
        pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }).then(res => {
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
  margin-left: -30px;
  margin-right: -30px;
}
.ant-card-body {
  padding-top: 0;
}
</style>
