<template>
  <a-card class="chart-card" :loading="loading" style="background:#F5F6FA; border-radius: 5px"
          :bordered="false">
    <div class="chart-card-header">
      <div class="meta">
        <span class="chart-card-title">{{ title }}</span>
        <span class="chart-card-action">
          <slot name="action"></slot>
        </span>
      </div>
      <div class="total">
        <span>{{ total }}</span>
      </div>
    </div>
    <div class="chart-card-content">
      <div class="content-fix">
        <slot></slot>
      </div>
    </div>
    <div class="chart-card-footer">
      <div class="field">
        <slot name="footer"></slot>
      </div>
    </div>
  </a-card>
</template>

<script>
export default {
  name: 'ChartCard',
  props: {
    title: {
      type: String,
      default: ''
    },
    total: {
      type: String,
      default: ''
    },
    loading: {
      type: Boolean,
      default: false
    }
  }
}
</script>

<style lang="less" scoped>
.chart-card {
  position: relative;
  margin-bottom: 20px;

  .chart-card-header {
    overflow: hidden;
    width: 100%;

    .meta {
      overflow: hidden;
      width: 100%;
      color: rgba(0, 0, 0, 0.45);
      font-size: 14px;
      line-height: 14px;

      .chart-card-title {
        display: inline-block;
        position: absolute;
        top: -24px;
        left: 0;
        padding: 6px;
        color: #fff;
        background-color: rgb(126, 114, 231);
        font-size: 14px;
        border-radius: 0 5px;
      }
    }

    .chart-card-action {
      cursor: pointer;
      position: absolute;
      top: 0;
      right: 0;
    }

    .total {
      overflow: hidden;
      text-overflow: ellipsis;
      word-break: break-all;
      white-space: nowrap;
      color: #000;
      margin-top: 4px;
      margin-bottom: 0;
      font-size: 30px;
      line-height: 38px;
      height: 38px;
    }
  }

  .chart-card-content {
    position: relative;
    margin-bottom: 12px;
    height: 46px;
    width: 100%;

    .content-fix {
      position: absolute;
      left: 0;
      bottom: 0;
      width: 100%;
    }
  }

  .chart-card-footer {

    > * {
      position: relative;
    }

    .field {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      margin: 0;
    }
  }

}


</style>
