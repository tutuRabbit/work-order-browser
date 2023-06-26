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
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <!-- 
        :loading="loading"
        :pagination="ipagination"
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
        // list: '/outentrust/outFixEntrustHistory/list',
        list: '/outentrust/outFixEntrustHistory/list',
        queryById: '/outentrust/outFixEntrustHistory/queryById'
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
      getAction(this.url.queryById, {
        id: val.id
      }).then(res => {
        if (res.success) {
          this.dataSource = res.result
          this.ipagination.total = res.result.total
        } else {
          this.dataSource = null
        }
      })
      // getAction(this.url.list, {
      //   outEntrustId: val.id,
      //   pageNo: this.ipagination.current,
      //   pageSize: this.ipagination.pageSize
      // }).then(res => {
      //   if (res.success) {
      //     this.dataSource = res.result.records
      //     this.ipagination.total = res.result.total
      //   } else {
      //     this.dataSource = null
      //   }
      // })
    }
  }
}
</script>
<style lang="less" scoped>
.ant-upload-list {
  height: auto;
  overflow-y: auto;
}
ul {
  list-style-type: none;
}
.list {
  display: flex;
  flex-wrap: wrap;
  flex: 1;
}
.item {
  width: 104px;
  height: 104px;
  position: relative;
  margin: 0 10px 10px 0;
  cursor: pointer;
}
.div {
  width: 100%;
  height: 100%;
  position: relative;
  padding: 5px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
}
</style>
