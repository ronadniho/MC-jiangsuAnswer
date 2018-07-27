<template>
  <div id="welcome">
    <div class="title-group">
      <h1>岗位技能竞赛</h1>
      <h2>“每日一练”在线答题</h2>
      <img src="../../assets/welcome-icon.png" alt="">
    </div>
    <div class="btn-group">
      <p>今天的天目论剑开始啦</p>
      <p>各位财侠接招吧</p>
      <div class="btn">
        <button @click="jump(1)">查看操作手册</button>
        <!--<a href="" download>查看操作手册</a>-->
        <button @click="jump(0)">进入答题</button>
      </div>
    </div>
  </div>
</template>

<script>
  import {config} from '../../../config/config';

  export default {
    name: "Welcome",
    data() {
      return {
        user_id: '',
      }
    },
    created: function () {
      console.log(config.productUrls)
      console.log(this.$route.query.user_id)
      if (this.$route.query.user_id) {
        this.user_id = this.$route.query.user_id;
      }
      else {
        mui.alert('用户不存在,请联系管理员', '', '返回', () => {
          window.opener = null;
          window.open('', '_self');
          window.close();

          // this.$router.go(-1)
        }, 'div');
      }
    },
    methods: {
      jump(val) {
        if (val) {
          console.log(config.productUrls)
          window.location.href = `http://172.80.72.17:8088/?furl=${config.productUrls}/JSSCZTOnlineAnswer/caozuoshouce/jscztmrylczsc.pdf
`;
        }
        else {
          this.$router.push({name: 'Main', params: {user_id: this.user_id}});
        }

      },
    }

  }
</script>

<style scoped lang="less">
  @import '../../variable';

  #welcome {
    position: fixed;
    left: 0;
    right: 0;
    top: 50%;
    bottom: 0;
    margin-top: -1081/2px;
    width: @max;
    height: 1081px;
    background: url(../../assets/welcome-bg.png) no-repeat;
    -webkit-background-size: 100%;
    background-size: 100%;
    .title-group {
      position: absolute;
      background-color: transparent;
      width: @max;
      text-align: center;
      top: 45px;
      .h-fn {
        color: @theme;
      }
      h1 {
        .h-fn;
        font-size: 60px;
        line-height: 96px;
      }
      h2 {
        .h-fn;
        font-size: 43px;
      }
      img{
        margin-top: 60px;
        width: 529px;
        height: 418px;
      }
    }

    .btn-group {
      position: absolute;
      bottom: 0;
      width: @max;
      text-align: center;
      p {
        font-size: 36px;
        line-height: 56px;
        color: #aaa;
      }
      p:nth-child(2) {
        padding-bottom: 123px;
      }
      .btn {
        button {
          background-color: @theme;
          color: @white;
          border: 0;
          border-radius: 35px;
          width: 246px;
          height: 70px;
          line-height: 70px;
          padding: 0;
        }
        button:first-child{
          margin-right:18px;

        }
        button:last-child{
          margin-left:18px;

        }
      }
    }
  }
</style>
