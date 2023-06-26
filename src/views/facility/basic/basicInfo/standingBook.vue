<template>
  <a-row type="flex" :gutter="16">
    <a-col :md="4" :sm="24">
      <standing-book-left v-model="currentOrgCode" />
    </a-col>
    <a-col :md="24 - 4" :sm="24">
      <standing-book-right v-model="currentOrgCode" />
    </a-col>
  </a-row>
</template>

<script>
import standingBookLeft from './standingBook/standingBookLeft'
import standingBookRight from './standingBook/standingBookRight'

export default {
  name: 'standingBook',
  components: { standingBookLeft, standingBookRight },
  data() {
    return {
      description: '设备台账页面',
      currentOrgCode: ''
    }
  },

  methods: {}
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
