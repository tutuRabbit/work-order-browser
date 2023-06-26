<template>
  <div :style="{ padding: '0 30px 32px 0' }">
    <h4 :style="{ marginBottom: '10px' }" class="h4s">{{ title }}</h4>
    <v-chart :forceFit="true" :height="height" :data="data" :scale="scale" :padding=" padding" :onClick="handleClick">
      <v-tooltip/>
      <v-legend/>
      <v-axis/>
      <v-bar position="type*bar"/>
      <v-line position="type*line" color="#2fc25b" :size="3"/>
    </v-chart>
  </div>
</template>

<script>
  import { ChartEventMixins } from './mixins/ChartMixins'

  export default {
    name: 'BarAndLine',
    mixins: [ChartEventMixins],
    props: {
      title: {
        type: String,
        default: ''
      },
      dataSource: {
        type: Array,
        default: () => [
          { type: '10:10', bar: 200, line: 577 },
          { type: '10:15', bar: 600, line: 354},
          { type: '10:20', bar: 200, line: 255},
          { type: '10:25', bar: 900, line: 445},
          { type: '10:30', bar: 200, line: 545},
          { type: '10:35', bar: 200, line:145},
          { type: '10:40', bar: 100, line: 44}
        ]
      },
      fields: {
        type: Array,
        default: () => ['bar', 'line']
      },
      // 别名，需要的格式：[{field:'name',alias:'姓名'}, {field:'sex',alias:'性别'}]
      aliases:{
        type: Array,
        default: () => ['1', '2']
      },
      height: {
        type: Number,
        default: 284
      }
    },
    data() {
      return {
        padding: { top:50, right:50, bottom:100, left:50 },
        scale: [{
          dataKey: 'bar',
          min: 0
        }, {
          dataKey: 'line',
          min: 0
        }]
      }
    },
    computed: {
      data() {
        return this.dataSource
      }
    }
  }
</script>

<style>
  .h4s{
    color: #fff;
    padding-top: 15px;
  }
</style>