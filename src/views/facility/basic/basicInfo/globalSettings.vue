<template>
  <a-card :bordered="false">
    <a-row :gutter="8">
      <a-col :span="12">
        <!-- 左上父 -->
        <div class="title_s">设备编码规则</div>
        <j-vxe-table
          row-number
          row-selection
          click-select-row
          highlight-current-row
          :columns="table1.columns"
          :dataSource="table1.dataSource"
          style="margin-top: 20px "
        />
        <!-- 左下子 -->
        <div class="title_s" style="margin-top: 20px ">维修单号规则</div>
        <j-vxe-table
          row-number
          row-selection
          click-select-row
          :loading="table3.loading"
          :columns="table3.columns"
          style="margin-top: 20px"
          :dataSource="table3.dataSource"
        />
      </a-col>
      <!-- 左侧父选择的数据展示在这里 -->
      <a-col :span="12">
        <!-- 右上父 -->
        <div class="title_s">报修单号规则</div>
        <j-vxe-table
          row-number
          row-selection
          click-select-row
          highlight-current-row
          :columns="table2.columns"
          :dataSource="table2.dataSource"
          style="margin-top: 20px"
        />
        <!-- 右下子 -->
        <div class="title_s" style="margin-top: 20px ">保养单号规则</div>
        <j-vxe-table
          row-number
          row-selection
          click-select-row
          :loading="table4.loading"
          :columns="table4.columns"
          :dataSource="table4.dataSource"
          style="margin-top: 20px"
        />
      </a-col>
    </a-row>
    <div
      style="text-align: center;
            padding: 10px;
            height: 50px;
            position: fixed;
            width: 100%;
            margin-left: -8%;"
    >
      <a-button type="primary" @click="onSubmit" medium>保 存</a-button>
    </div>
  </a-card>
</template>

<script>
import { getAction, postAction } from '@api/manage'
import { JVXETypes } from '@/components/jeecg/JVxeTable'

export default {
  name: 'globalSettings',
  data() {
    return {
      table1: {
        // 是否正在加载
        loading: false,
        // 最后选中的行
        lastRow: null,
        // 数据源，控制表格的数据
        dataSource: [],
        // 列配置，控制表格显示的列
        columns: [
          {
            // 字段key，跟后台数据的字段名匹配
            key: 'codingRules',
            // 列的标题
            title: '编码规则',
            // 如果加上了该属性，就代表当前单元格是可编辑的，type就是表单的类型，input就是简单的输入框
            type: JVXETypes.select,
            dictCode: 'coding_rules_equipment',
            options: []
          },
          { key: 'codingContent', title: '规则内容', type: JVXETypes.input }
        ]
      },
      table2: {
        loading: false,
        dataSource: [],
        columns: [
          {
            key: 'codingRules',
            title: '编码规则',
            type: JVXETypes.select,
            dictCode: 'coding_rules_fix',
            options: []
          },
          { key: 'codingContent', title: '规则内容', type: JVXETypes.input }
        ]
      },
      table3: {
        loading: false,
        dataSource: [],
        columns: [
          {
            key: 'codingRules',
            title: '编码规则',
            type: JVXETypes.select,
            dictCode: 'coding_rules_requir',
            options: []
          },
          { key: 'codingContent', title: '规则内容', type: JVXETypes.input }
        ]
      },
      table4: {
        loading: false,
        dataSource: [],
        columns: [
          {
            key: 'codingRules',
            title: '编码规则',
            type: JVXETypes.select,
            dictCode: 'coding_rules_requir',
            options: []
          },
          { key: 'codingContent', title: '规则内容', type: JVXETypes.input }
        ]
      },
      // 查询url地址
      url: {
        getData: '/equipemnt/equipmentCodingRules/list',
        update: '/equipemnt/equipmentCodingRules/update'
      }
    }
  },
  created() {
    this.loadDatas()
  },
  methods: {
    onSubmit() {
      let that = this
      let list = new Array()
      that.table1.dataSource.forEach(item => {
        list.push(item)
      })
      that.table2.dataSource.forEach(item => {
        list.push(item)
      })
      that.table3.dataSource.forEach(item => {
        list.push(item)
      })
      that.table4.dataSource.forEach(item => {
        list.push(item)
      })
      postAction(this.url.update, list).then(res => {
        if (res.success) {
          if (res.code == 200) {
            this.loadDatas()
            this.$message.success(res.message)
          }
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    loadDatas() {
      // 调用查询数据接口
      this.table1.loading = true
      this.table1.dataSource = []
      this.table2.dataSource = []
      this.table3.dataSource = []
      this.table4.dataSource = []
      getAction(this.url.getData)
        .then(res => {
          if (res.success) {
            res.result.forEach(val => {
              switch (val.codingType) {
                case '1':
                  // 设备编码规则
                  this.table1.dataSource.push(val)
                  break
                case '2':
                  // 维修单号规则
                  this.table2.dataSource.push(val)
                  break
                case '3':
                  // 报修单号规则
                  this.table3.dataSource.push(val)
                  break
                default:
                  // 保养单号规则
                  this.table4.dataSource.push(val)
              }
            })
          } else {
            this.$error({ title: '全局配置主表查询失败', content: res.message })
          }
        })
        .finally(() => {
          // 这里是无论成功或失败都会执行的方法，在这里关闭loading
          this.table1.loading = false
        })
    }
  }
}
</script>

<style scoped>
.title_s {
  font-size: 18px;
  color: #000;
  font-weight: 600;
  margin-top: 2px;
  background-color: #efefefd4;
}
</style>
