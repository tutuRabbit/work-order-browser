<template>
  <a-card :loading="cardLoading" :bordered="false" style="height: 100%;">
    <a-spin :spinning="loading">
      <div
        style="color: #484848;
              font-size: 18px;
              font-weight: 600;
"
      >
        设备类别
      </div>
      <a-tree
        :replaceFields="replaceFields"
        :loadedKeys="loadedKeys"
        @select="handleTreeSelect"
        @expand="onExpand"
        @load="loadDept"
        :expandedKeys="expandedKeys"
        :load-data="onLoadDeptData"
        :tree-data="treeDeptData"
      />
    </a-spin>
  </a-card>
</template>

<script>
import { queryDepartTreeList } from '@/api/api'
import { getAction } from '@/api/manage'

export default {
  name: 'standingBookLeft',
  props: ['value'],
  data() {
    return {
      cardLoading: true,
      loading: false,
      treeDataSource: [],
      selectedKeys: [],
      expandedKeys: [],
      url: {
        list: '/sys/category/loadTreeData',
        childList: '/sys/category/childList'
      },
      //异步加载
      loadedKeys: [],
      expandedKeys: [],
      isShowAll: false, //非监管部门是否展示
      treeDeptData: [],
      replaceFields: {
        children: 'children',
        title: 'name',
        key: 'id'
      } // 树数据结构匹配替换映射
      // end
    }
  },
  created() {
    this.queryTreeData()
    this.RegionDeptTreeByParent()
  },
  methods: {
    // 异步加载数据
    //loadedKeys	（受控）已经加载的节点，需要配合 loadData 使用
    //	节点加载完毕时触发
    loadDept(loadedKeys) {
      this.loadedKeys = loadedKeys
    },

    //展开/收起节点时触发
    onExpand(keys) {
      // expandedKeys(.sync)	（受控）展开指定的树节点
      this.expandedKeys = keys
    },
    // 异步加载数据
    onLoadDeptData(treeNode) {
      // console.log('treeNode', treeNode)
      // console.log('treeNode', treeNode.dataRef)
      // console.log('id', treeNode.dataRef.id)
      // console.log('pid', treeNode.dataRef.pid)
      return new Promise(resolve => {
        setTimeout(() => {
          let pm = {
            pid: treeNode.dataRef.id,
            order: 'desc',
            pageNo: 1,
            pageSize: 10
          }
          getAction(this.url.childList, pm).then(res => {
            treeNode.dataRef.children = res.result.map(item => {
              if (item.hasChild == '1') {
                item.isLeaf = false
              } else {
                item.isLeaf = true
              }
              return item
            })
            this.treeDeptData = [...this.treeDeptData]
            resolve()
          }, 1000)
        })
      })
    },

    //获取树结构
    RegionDeptTreeByParent() {
      this.loadedKeys = []
      this.expandedKeys = []
      this.commonRequestThen(queryDepartTreeList())
    },

    // end
    queryTreeData(keyword) {
      this.commonRequestThen(
        queryDepartTreeList({
          departName: keyword ? keyword : undefined
        })
      )
    },
    handleTreeSelect(selectedKeys, event) {
      if (selectedKeys.length > 0) {
        if (this.selectedKeys[0] !== selectedKeys[0]) {
          this.selectedKeys = [selectedKeys[0]]
          let equipmentType = event.node.dataRef.id
          this.emitInput(equipmentType)
        }
      } else {
        this.selectedKeys = []
        this.emitInput('')
      }
    },
    handleExpand(expanded, record, e) {
      console.log('e', e)
      console.log('record', record)
      console.log('record', record.id)
      console.log('expanded', record.expanded)
      console.log('expanded', expanded)
      // 判断是否是展开状态
      if (record.expanded) {
        // this.expandedRowKeys.push(record.id)
        if (record.children.length > 0 && record.children[0].isLoading === true) {
          let params = this.getQueryParams() //查询条件
          params[this.pidField] = record.id
          getAction(this.url.childList, params).then(res => {
            if (res.success) {
              if (res.result && res.result.length > 0) {
                record.children = this.getDataByResult(res.result)
                this.dataSource = [...this.dataSource]
              } else {
                record.children = ''
                record.hasChildrenField = '0'
              }
            } else {
              this.$message.warning(res.message)
            }
          })
        }
      } else {
        let keyIndex = this.expandedRowKeys.indexOf(record.id)
        if (keyIndex >= 0) {
          this.expandedRowKeys.splice(keyIndex, 1)
        }
      }
    },
    emitInput(equipmentType) {
      this.$emit('input', equipmentType)
    },

    commonRequestThen(promise) {
      console.log('promise', promise)
      this.loading = true
      let url = '/sys/category/loadTreeData'
      let param = {
        pid: '1549960432278106114',
        order: 'desc',
        pageNo: 1,
        pageSize: 10
      }
      getAction(this.url.childList, param)
        // getAction(url, param)
        .then(res => {
          if (res.success && res.result) {
            for (let i of res.result) {
              if (i.hasChild == '1') {
                i.isLeaf = false
              } else {
                i.isLeaf = true
              }
            }
            this.treeDataSource = [...res.result]
            this.treeDeptData = [...res.result]
          } else {
            console.log('树一级节点查询结果-else', res)
          }
        })
        .finally(() => {
          this.loading = false
          this.cardLoading = false
        })
    }
  }
}
</script>

<style scoped></style>
