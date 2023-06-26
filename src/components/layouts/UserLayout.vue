<template>
  <div id="userLayout" :class="['user-layout-wrapper', device]">
    <div class="container">
      <div class="login-pic">
        <img :src="require('@/assets/login-pic.png')" alt="" style="height: 100%; width: 100%">
      </div>
      <!--      <div class="top">-->
      <!--        <div class="header">-->
      <!--          <a href="/">-->
      <!--            &lt;!&ndash;            <img src="~@/assets/logo.svg" class="logo" alt="logo" />&ndash;&gt;-->
      <!--            <span class="title">SHE BEI YUN WEI GUAN LI</span>-->
      <!--          </a>-->
      <!--        </div>-->
      <!--        <div class="desc">-->
      <!--          设备运维管理系统-->
      <!--          &lt;!&ndash; Jeecg Boot 是中国最具影响力的 企业级 低代码平台 &ndash;&gt;-->
      <!--        </div>-->
      <!--      </div>-->
      <div class="login-win">
        <div class="login-title">
          <p>请登录</p>
          <div></div>
        </div>
      </div>
      <div class="tips">
        <p>建议使用1920*1080分辨率/谷歌浏览器80.0以上版本访问达到最佳效果</p>
      </div>
      <route-view></route-view>

      <!-- <div class="footer">
        <div class="links">
          <a href="http://doc.jeecg.com" target="_blank">帮助</a>
          <a href="https://github.com/zhangdaiscott/jeecg-boot" target="_blank">隐私</a>
          <a href="https://github.com/zhangdaiscott/jeecg-boot/blob/master/LICENSE" target="_blank">条款</a>
        </div>
        <div class="copyright">
          Copyright &copy; 2019 <a href="http://www.jeecg.com" target="_blank">JEECG开源社区</a> 出品
        </div>
      </div> -->
    </div>
  </div>
</template>

<script>
import RouteView from '@/components/layouts/RouteView'
import {mixinDevice} from '@/utils/mixin.js'

export default {
  name: 'UserLayout',
  components: {RouteView},
  mixins: [mixinDevice],
  data() {
    return {}
  },
  mounted() {
    document.body.classList.add('userLayout')
  },
  beforeDestroy() {
    document.body.classList.remove('userLayout')
  }
}
</script>

<style lang="less" scoped>
#userLayout.user-layout-wrapper {
  height: 100%;

  &.mobile {
    .container {
      .main {
        max-width: 368px;
        width: 98%;
      }
    }
  }

  .container {
    width: 100%;
    min-height: 100%;
    background: url(~@/./assets/bg.png) no-repeat 100% 100%;
    background-size: 100% 100%;
    padding: 110px 0 144px;
    position: relative;

    a {
      text-decoration: none;
    }

    .login-pic {
      position: absolute;
      left: 320px;
      top: 165px;
      width: 37vw;
      height: 67vh;
    }

    .login-win {
      position: absolute;
      right: 320px;
      top: 165px;
      width: 30vw;
      height: 67vh;
      background: #fff;
      border-radius: 0 10px 10px 0;

      .login-title {
        position: absolute;
        left: 50%;
        top: 20%;
        line-height: 16px;
        color: #1F2028;
        transform: translateX(-50%);

        p {
          font-size: 20px;
          font-weight: bold;

        }

        div {
          position: absolute;
          top: 13px;
          left: -10px;
          width: 80px;
          height: 10px;
          background: #3355FF;
          opacity: 0.5;
        }
      }
    }

    .tips {
      position: absolute;
      left: 50%;
      bottom: 2.5%;
      transform: translateX(-50%);
      font-size: 14px;
      color: #2D69E0;
      line-height: 16px;


      p {
        width: 100%;
        text-align: center;
      }
    }


    .top {
      text-align: center;

      .header {
        height: 44px;
        line-height: 44px;

        .badge {
          position: absolute;
          display: inline-block;
          line-height: 1;
          vertical-align: middle;
          margin-left: -12px;
          margin-top: -10px;
          opacity: 0.8;
        }

        .logo {
          height: 44px;
          vertical-align: top;
          margin-right: 16px;
          border-style: none;
        }

        .title {
          font-size: 33px;
          color: rgba(0, 0, 0, 0.85);
          font-family: 'Chinese Quote', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB',
          'Microsoft YaHei', 'Helvetica Neue', Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji',
          'Segoe UI Symbol';
          font-weight: 600;
          position: relative;
          top: 2px;
        }
      }

      .desc {
        font-size: 14px;
        color: rgba(0, 0, 0, 0.45);
        margin-top: 12px;
        margin-bottom: 40px;
      }
    }

    .main {
      min-width: 260px;
      width: 368px;
      margin: 0 auto;
    }

    .footer {
      position: absolute;
      width: 100%;
      bottom: 0;
      padding: 0 16px;
      margin: 48px 0 24px;
      text-align: center;

      .links {
        margin-bottom: 8px;
        font-size: 14px;

        a {
          color: rgba(0, 0, 0, 0.45);
          transition: all 0.3s;

          &:not(:last-child) {
            margin-right: 40px;
          }
        }
      }

      .copyright {
        color: rgba(0, 0, 0, 0.45);
        font-size: 14px;
      }
    }
  }
}
</style>
