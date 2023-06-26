<template>
  <div :style="{ padding: '0 0 32px 32px' }">
    <h4 :style="{ marginBottom: '10px' }" class="h4s">{{ title }}</h4>
    <v-chart :forceFit="true" :height="height" :data="dataSource" :scale="scale" :padding="padding">
      <v-tooltip/>
      <v-axis/>
      <v-bar position="x*y"/>
    </v-chart>
  </div>
</template>

<script>
  import { triggerWindowResizeEvent } from '@/utils/util'

  export default {
    name: 'Bar',
    props: {
      dataSource: {
        type: Array,
        required: true
      },
      yaxisText: {
        type: String,
        default: 'y'
      },
      title: {
        type: String,
        default: ''
      },
      height: {
        type: Number,
        default: 184
      }
    },
    data() {
      return { padding: ['auto', 'auto', '40', '50'] }
    },
    computed: {
      scale() {
        return [{
          dataKey: 'y',
          alias: this.yaxisText
        }]
      }
    },
    mounted() {
      triggerWindowResizeEvent()
    }
  }
</script>
<style>
  .h4s{
    color: #fff;
    padding-top: 15px;
  }
</style>