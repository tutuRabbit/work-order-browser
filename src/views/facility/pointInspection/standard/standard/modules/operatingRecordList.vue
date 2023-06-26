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
        bordered
        rowKey="id"
        :columns="columns"
        :scroll="{ y: 180 }"
        :dataSource="dataSource"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
        :pagination="false"
      >
        <!-- 
        :loading="loading"
        :pagination="ipagination"
       -->
        <!-- <template v-slot:recordingMethod="props">
          <span>{{
            props.row.recordingMethod == '0'
              ? '单选'
              : props.row.recordingMethod == '1'
              ? '多选'
              : props.row.recordingMethod == '2'
              ? '数字'
              : '文本'
          }}</span>
        </template> -->
        <template slot="recordingMethod" slot-scope="text, record">
          <span>{{
            record.recordingMethod == '0'
              ? '单选'
              : record.recordingMethod == '1'
              ? '多选'
              : record.recordingMethod == '2'
              ? '数字'
              : '文本'
          }}</span>
        </template>
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
          title: '检查项',
          align: 'center',
          width: 200,
          dataIndex: 'name'
        },
        {
          title: '结果类型',
          dataIndex: 'recordingMethod',
          align: 'center',
          scopedSlots: { customRender: 'recordingMethod' }
        },
        {
          title: '上限',
          align: 'center',
          dataIndex: 'upperLimit'
        },
        {
          title: '下限',
          align: 'center',
          dataIndex: 'lowerLimit'
        },
        {
          title: '标准图片',
          align: 'center',
          dataIndex: 'operationResult'
        }
      ],
      dataSource: [],
      url: {
        // list: '/outentrust/outFixEntrustHistory/list',
        queryById: '/checkout/pointpatrolstandard/findById'
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
        if (res.code == 200) {
          this.dataSource = res.data.inspectionstandardList
          this.ipagination.total = res.data.total
        } else {
          this.dataSource = null
        }
      })
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
