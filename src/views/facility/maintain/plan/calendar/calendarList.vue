<template>
  <a-card :bordered="false">
    <a-calendar @panelChange="onPanelChange" :header-render="headerRender" @select="select">
      <ul slot="dateCellRender" slot-scope="value">
        <li v-for="(item, i) in getListData(value)" :key="i" class="item-list" @click="handleLink(item)">
          <span>{{ item.title ? item.title : '暂无名称' }}</span>
        </li>
      </ul>
      <ul slot="monthCellRender" slot-scope="value">
        <li v-if="getYearData(value)" class="notes-month item-list" @click="handleLink(value)">
          <section>{{ getYearData(value) }}</section>
        </li>
      </ul>
    </a-calendar>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import moment from 'moment' //时间格式化插件
import { getAction } from '../../../../../api/manage'

export default {
  name: 'calendarList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {},
  data() {
    return {
      description: '计划日历管理页面',
      url: {
        list: '/keepOrder/tabKeepOrder/GetMainPlan'
      },
      testList: [],
      yearList: [],
      modeStatus: '',
      searchList: {}
    }
  },
  beforeMount() {
    this.getSuperFieldList('2022-01-01', '2022-12-31')
  },
  computed: {},
  methods: {
    // 头部
    headerRender({ value, type, onChange, onTypeChange }) {
      const start = 0
      const end = 12
      const monthOptions = []

      const current = value.clone()
      const localeData = value.localeData()
      const months = []
      for (let i = 0; i < 12; i++) {
        current.month(i)
        months.push(localeData.monthsShort(current))
      }

      for (let index = start; index < end; index++) {
        monthOptions.push(
          <a-select-option class="month-item" key={`${index}`}>
            {months[index]}
          </a-select-option>
        )
      }
      const month = value.month()

      const year = value.year()
      const options = []
      for (let i = year - 10; i < year + 10; i += 1) {
        options.push(
          <a-select-option key={i} value={i} class="year-item">
            {i + '年'}
          </a-select-option>
        )
      }
      // 显示今天
      const showToday = () => {
        const today = moment(new Date())
        onChange(today)
      }
      // <a-radio-group size="small" onChange={e => onTypeChange(e.target.value)} value={type}>
      //            <a-radio-button value="month">月</a-radio-button>
      //            <a-radio-button value="week">周</a-radio-button>
      //            <a-radio-button value="day">天</a-radio-button>
      //         </a-radio-group>
      return (
        <div style={{ padding: '10px' }}>
          <a-row type="flex" justify="space-between">
            <a-col>
              <a-radio-group size="small" onChange={e => onTypeChange(e.target.value)} value={type}>
                <a-radio-button value="month">月</a-radio-button>
                <a-radio-button value="year">年</a-radio-button>
              </a-radio-group>
            </a-col>
            <a-col>
              <a-select
                size="small"
                dropdownMatchSelectWidth={false}
                onChange={newYear => {
                  const now = value.clone().year(newYear)
                  onChange(now)
                }}
                value={String(year) + '年'}
              >
                {options}
              </a-select>
            </a-col>
            <a-col>
              <a-select
                size="small"
                dropdownMatchSelectWidth={false}
                value={String(month)}
                onChange={selectedMonth => {
                  const newValue = value.clone()
                  newValue.month(parseInt(selectedMonth, 10))
                  onChange(newValue)
                }}
              >
                {monthOptions}
              </a-select>
            </a-col>
            <a-col>
              <a-button type="primary" onClick={() => showToday()}>
                返回今日
              </a-button>
            </a-col>
          </a-row>
        </div>
      )
    },
    // end
    handleLink(val) {
      console.log('点点点', val)
      console.log('跳转need的id', val.id)
      this.$router.push({ path: '/facility/maintain/plan/plan/TabKeepOrderList', query: { index: val.id } })
    },
    getListData(value) {
      let listData = []
      //遍历数组
      this.testList.forEach(e => {
        //让数组的date与遍历到日历的那天的日期值相匹配
        if (e.startTime === value.format('YYYY-MM-DD')) {
          listData.push(e)
        }
      })
      return listData || []
    },
    // 日期面板变化
    onPanelChange(value, mode) {
      console.log('日期面板变化mode:', mode)
      console.log(moment(value).format('YYYY-MM-DD'))
      let list = moment(value).format('YYYY-MM-DD')
      if (mode == 'year') {
        this.modeStatus = 'year'
        let start = list.slice(0, 4)
        let pm = {
          startTime: start + '-01-01',
          endTime: start + '-12-31'
        }
        this.yearList = []
        getAction(this.url.list, pm).then(res => {
          if (res.success) {
            this.yearList = res.result
            console.log('this.yearList', this.yearList)
          }
        })
      } else {
        this.modeStatus = 'month'
      }
    },
    // 点击选择日期
    select(value) {
      console.log(value)
      console.log(moment(value).format('YYYY-MM-DD'))
    },

    // 获取相同项为新数组
    dataDeal(data) {
      console.log('获取相同项为新数组', data)
      var listArr = []
      data.forEach(function(item, index) {
        for (var i = 0; i < listArr.length; i++) {
          // 对比相同的字段key，相同放入对应的数组
          if (listArr[i].startTime === item.startTime) {
            listArr[i].listInfo.push({
              num: item.num
            })
            return
          }
        }
        // 第一次对比没有参照，放入参照
        listArr.push({
          startTime: item.startTime,
          listInfo: [
            {
              num: item.num
            }
          ]
        })
      })
      listArr.forEach(item => {
        let sum = 0
        item.listInfo.forEach(items => {
          sum += items.num
          item.sum = sum
        })
      })
      console.log('listArr', listArr)
      return listArr
    },
    // 年份面板显示内容
    getYearData(value) {
      var list = moment(value).format('YYYY-MM-DD')
      // console.log('list', list)
      // this.yearList.forEach(item => {
      //   var endTime = item.endTime.slice(5, 2)
      //   var endList = list.slice(5, 2)
      //   if (endList == endTime) {
      //   } else {
      //     return item
      //   }
      // })
      // if(this.yearList){

      // }
      return list
    },
    getSuperFieldList(start, end) {
      let pm = {
        startTime: start,
        endTime: end
      }
      this.testList = []
      getAction(this.url.list, pm).then(res => {
        if (res.success) {
          this.testList = res.result
          console.log('this.testList', this.testList)
        }
      })
    },
    // 回到今天
    showToday() {
      const today = moment(new Date())
      onChange(today)
    },
    onChange(newYear) {
      alert(newYear)
      console.log('newYear', newYear)
      const now = value.clone().year(newYear)
      onChange(now)
    }
  }
}
</script>
<style>
/* .ant-radio-group {
  display: none;
} */
.month-item {
  /* background-color: aqua; */
}
.item-list {
  list-style: none;
  width: 100%;
  padding: 5px 0;
  margin-bottom: 5px;
  color: #fff;
  border-radius: 5px;
  background-color: #fb7507;
  text-align: center;
}
</style>
