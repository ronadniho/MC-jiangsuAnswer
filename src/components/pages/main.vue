<template>
  <div class="main">
    <!--<Header-->
    <!--:title="title"-->
    <!--:href="href"-->
    <!--:jumpState="jumpState"-->
    <!--:alert="alert"-->
    <!--@child="warning"-->
    <!--/>-->
    <div class="main-content">

      <div class="action flex">

        <div class="user">
          <div class="name"> 您好，{{userInfo.NAME}}</div>
          <div class="time">{{date}}</div>
        </div>
        <div class="tabber flex">
          <div class="tab-item" :class="[current==0?'active':'']" @click="toggleTab(0)">积分</div>
          <div class="tab-item" :class="[current==1?'active':'']" @click="toggleTab(1)">排行榜</div>
        </div>
      </div>

      <div class="content">
        <!--积分-->
        <div class="tab-ctx" v-if="!current">
          <div class="bg">
            <div class="bg-pos" @click="jump">
              <h2>总积分</h2>
              <h1>{{userInfo.INTEGRAL?userInfo.INTEGRAL:0}}</h1>
              <div><img src="../../assets/sb.png" alt=""></div>
            </div>
          </div>
          <!--
                    <div class="bg">
                      <div class="bg-pos" @click="jump">
                        &lt;!&ndash;<WaterPolo :num="num" v-if="state"/>&ndash;&gt;
                        <span>{{userInfo.INTEGRAL}}</span>
                      </div>
                    </div>
          -->
        </div>
        <!--排名-->
        <div class="tab-ctx" v-if="current">
          <div class="wrapper-header flex">
            <div class="grid-header">序号</div>
            <div class="grid-header">姓名</div>
            <div class="grid-header">处室</div>
            <div class="grid-header">积分</div>
            <div class="grid-header">排名</div>
          </div>
          <div class="wrapper-content active flex" v-if="userRanking&&my">
            <div class="grid-item">{{userRanking.ROWNUM}}</div>
            <div class="grid-item">{{userRanking.NAME}}</div>
            <div class="grid-item">{{userRanking.ACADEMY}}</div>
            <div class="grid-item">{{userRanking.INTEGRAL}}</div>
            <div class="grid-item">{{userRanking.ROWNO}}</div>
          </div>
          <div class="scroller-con" :style="{'height':h}">
            <scroller
              :on-refresh="refresh"
              :on-infinite="infinite"
              :noDataText="'没有数据'"
              ref="my_scroller"
            >
              <!--<div style="height: 1px;"></div>-->
              <!-- content goes here -->
              <div id="wrapper-list" class="wrapper-list">

                <!--item-->
                <div class="wrapper-content flex"
                     v-for="(val,index) in rankingList"
                     :class="[val.hasClass?'active':'']">
                  <div class="grid-item">{{index+1}}</div>
                  <div class="grid-item">{{val.NAME}}</div>
                  <div class="grid-item">{{val.ACADEMY}}</div>
                  <div class="grid-item">{{val.INTEGRAL}}</div>
                  <div class="grid-item">{{val.ranking}}</div>
                  <!--<div class="grid-item">{{val.ROWNUM}}</div>-->
                </div>
                <!--<div class="wrapper-content active" v-if="userRanking">
                  <div class="grid-item">{{userRanking.ROWNO}}</div>
                  <div class="grid-item">{{userRanking.NAME}}</div>
                  <div class="grid-item">{{userRanking.ACADEMY}}</div>
                  <div class="grid-item">{{userRanking.INTEGRAL}}</div>
                </div>
                <div class="wrapper-content active" v-if="userRanking">
                  <div class="grid-item">{{userRanking.ROWNO}}</div>
                  <div class="grid-item">{{userRanking.NAME}}</div>
                  <div class="grid-item">{{userRanking.ACADEMY}}</div>
                  <div class="grid-item">{{userRanking.INTEGRAL}}</div>
                </div>
                <div class="wrapper-content active" v-if="userRanking">
                  <div class="grid-item">{{userRanking.ROWNO}}</div>
                  <div class="grid-item">{{userRanking.NAME}}</div>
                  <div class="grid-item">{{userRanking.ACADEMY}}</div>
                  <div class="grid-item">{{userRanking.INTEGRAL}}</div>
                </div>-->

              </div>


            </scroller>

          </div>


        </div>

      </div>


      <div class="btn-group">
        <button class="btn start" @click="goLink('start')">开始答题</button>
        <button class="btn read" @click="goLink('read')">查看题库</button>
      </div>
    </div>
    <!--loading-->
    <Loading v-if="loading"/>
  </div>
</template>

<script>
  import Header from '../common/Header';
  import Loading from '../common/Loading';
  import WaterPolo from '../common/WaterPolo';
  import {request} from '../../api/common';
  import {times} from '../../api/time';
  import {__GlobalInfo} from "../../api/config";

  import $ from 'jquery';
  import {__BASE64} from '../../api/base64'

  export default {
    name: 'Main',
    components: {
      Header,
      Loading,
      WaterPolo,
    },
    data() {
      return {
        title: '财政厅岗位技能竞赛',
        href: -1,
        jumpState: false,
        // href: 'login',
        // jumpState: true,
        alert: true,
        current: 0,
        num: 40,
        score: 80,
        userInfo: {},
        user_name: '',
        user_id: '',
        loading: false,
        rankingList: [],//排行集合
        userRanking: {},//个人排名
        rankingItemH: 68,//每条高度
        PAGESTART: 0,
        PAGECOUNT: 9,//每次加载十条
        loadList: true,
        h: '',
        my: true,
        date: '',
      }
    },
    beforeRouteEnter(to, from, next) {
      console.log(to);
      console.log(from);
      next(vm => {
        /*if (!vm.global.isLogin) {
          // console.log(vm.$router);
          // vm.$router.push({path: "login"});
        } else {
          // next();
        }*/
        next();
      });
    },
    beforeRouteLeave(to, from, next) {
      console.log(to.path);
      console.log(from.path);
      var alerts = e => {
        console.log(e);
        if (!e.index) {
          next();
        }
        else {
          this.global.isLogin = true;
          next(false);
        }
      };

      if (to.path == "/login" && this.global.isLogin) {
        this.global.isLogin = false;
        // next();
        // this.$router.push({ path: "login" });
        // mui.confirm("是否退出登录", "", ["是", "否"], alerts, "div");
      } else {
        next();
      }
    },
    created: function () {
      this.date = times();
      // if(!this.$route.query.user_id){
      //   mui.alert('服务器繁忙', '', '返回', ()=>{
      //     this.$router.go(-1);
      //   }, 'div');
      // }
      if (this.$route.params.user_id) {
        this.user_id = this.$route.params.user_id;
      }
      else {
        mui.alert('用户不存在,请联系管理员', '', '返回', () => {
          window.opener = null;
          window.open('', '_self');
          window.close();

          // this.$router.go(-1)
        }, 'div');
      }
      /*
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
      */

      // this.user_id = '111111';
      this.PAGESTART = -1;
      this.postLogin();
      // console.log(this.global.isLogin)

    },
    mounted: function () {
      var htmlFS = $('html').css('fontSize').replace('px', '');
      this.h = ((this.PAGECOUNT * this.rankingItemH) / 2 / htmlFS) + 'rem';
      $('.scroller-con').css('height', this.h)
      // $('._v-container').css('height', this.h + 'rem')
    },
    methods: {
      jump() {
        this.$router.push({name: 'Detail', params: {user_id: this.user_id}});
      },
      postLogin() {
        // this.loading = true;

        request.getServerData(
          {
            user_id: this.user_id,
          },
          "onlineExamination.action.examinationAction[getUserInfo]",
          true,
          (result) => {
            console.log(result)
            if (result.userInfo) {
              this.userInfo = result.userInfo;
              this.user_name = result.userInfo.NAME;
              sessionStorage.setItem(__GlobalInfo.sessionKey.userInfo, JSON.stringify(result.userInfo));
            }
          });
      },
      // postLogin() {
      //   alert('请求前')
      //   // this.loading = true;
      //   request.getServerData(
      //     {
      //       user_id: this.user_id,
      //     },
      //     "onlineExamination.action.examinationAction[getUserInfo]",
      //     true,
      //     (result) => {
      //       alert(JSON.stringify(result))
      //       console.log(result)
      //       if (result.userInfo) {
      //         this.userInfo = result.userInfo;
      //       }
      //     });
      // },
      warning() {
        var self = this;
        this.$router.push({path: 'login'});
        //mui.confirm('是否退出登录', '退出', ['是', '否'], self.alerts, 'div');
      },
      alerts(e) {
        !e.index && this.$router.push({path: 'login'});
      },
      refresh(done) {
        setTimeout(() => {
          this.PAGESTART = -1;
          this.loadList = true;
          this.rankingList = [];
          this.postList(done);
        }, 500)
      },
      infinite(done) {
        setTimeout(() => {
          console.log('加载')

          if (this.loadList) {
            this.postList(done)
          }
          else {
            done(true);
          }
          ;
          // this.postList(done);
        }, 500)
      },
      postList(fn) {
        this.PAGESTART++;
        // this.loading = true;
        request.getServerData(
          {
            PAGESTART: this.PAGESTART * this.PAGECOUNT,
            PAGECOUNT: this.PAGECOUNT,
            user_id: this.user_id
          },
          "onlineExamination.action.examinationAction[getRankingList]",
          true,
          (result) => {
            console.log(result)
            console.log('PAGESTART:' + this.PAGESTART)
            if (result.rankingList) {

              if (this.PAGESTART) {
                if (result.rankingList.length) {//如果是数组//一条以上是数组
                  for (var i = 0, rankingList = result.rankingList; i < rankingList.length; i++) {
                    if (result.userRanking) {
                      if (rankingList[i].ID == result.userRanking.ID) {
                        rankingList[i].hasClass = 1;
                        this.my = false;
                      }
                      else {
                        rankingList[i].hasClass = 0;
                      }
                    }

                    //INTEGRAL 积分
                    //ranking 排名
                    var lastInfo = this.rankingList[this.rankingList.length - 1];
                    console.log(lastInfo)
                    if (i) {
                      if (rankingList[i].INTEGRAL == rankingList[i - 1].INTEGRAL) {
                        rankingList[i].ranking = rankingList[i - 1].ranking;
                      }
                      else {
                        rankingList[i].ranking = (rankingList[i - 1].ranking) + 1;
                      }
                    }
                    else {//第一次
                      if(lastInfo.INTEGRAL>rankingList[i].INTEGRAL){
                        rankingList[i].ranking = i + 1+lastInfo.ranking;
                      }
                      else if(lastInfo.INTEGRAL<rankingList[i].INTEGRAL){
                        rankingList[i].ranking = i +lastInfo.ranking;
                      }
                      else{
                        rankingList[i].ranking = lastInfo.ranking;
                      }
                    }

                    this.rankingList.push(rankingList[i]);
                  }
                }
                else {//否则是对象//一条数据是对象
                  if (result.userRanking) {
                    if (result.rankingList.ID == result.userRanking.ID) {
                      result.rankingList.hasClass = 1;
                      this.my = false;
                    }
                    else {
                      result.rankingList.hasClass = 0;
                    }
                    this.rankingList.push(result.rankingList);
                  }
                }
              }


              else {
                this.userRanking = result.userRanking;
                for (var i = 0, rankingList = result.rankingList; i < rankingList.length; i++) {
                  if (result.userRanking) {
                    if (rankingList[i].ID == result.userRanking.ID) {
                      rankingList[i].hasClass = 1;
                      this.my = false;
                    }
                    else {
                      rankingList[i].hasClass = 0;
                    }

                    if (i) {
                      if (rankingList[i].INTEGRAL == rankingList[i - 1].INTEGRAL) {
                        rankingList[i].ranking = rankingList[i - 1].ranking;
                      }
                      else {
                        rankingList[i].ranking = (rankingList[i - 1].ranking) + 1;
                      }
                    }
                    else {
                      rankingList[i].ranking = i + 1;
                    }

                  }
                  else {

                    if (i) {
                      if (rankingList[i].INTEGRAL == rankingList[i - 1].INTEGRAL) {
                        rankingList[i].ranking = rankingList[i - 1].ranking;
                      }
                      else {
                        rankingList[i].ranking = (rankingList[i - 1].ranking) + 1;
                      }
                    }
                    else {
                      rankingList[i].ranking = i + 1;
                    }


                    this.my = false;
                  }
                }
                this.rankingList = rankingList;
              }
              /*for (var i = 0, rankingList = this.rankingList; i < rankingList.length; i++) {
                if (result.userRanking) {
                  if (rankingList[i].ID == result.userRanking.ID) {
                    rankingList[i].hasClass = 1;
                    this.my = false;
                  }
                  else {
                    rankingList[i].hasClass = 0;
                  }
                  if (i) {
                    if (rankingList[i].INTEGRAL == rankingList[i - 1].INTEGRAL) {
                      rankingList[i].ranking = rankingList[i - 1].ranking;
                    }
                    else {
                      rankingList[i].ranking = (rankingList[i - 1].ranking) + 1;
                    }
                  }
                  else {
                    rankingList[i].ranking = i + 1;
                  }
                }
                else {
                  this.my = false;
                }
              }*/
              fn();
            }
            else {
              this.my = false;
              this.loadList = false;
              fn(true);
            }


            // this.loading = false;
          });
      },
      toggleTab(e) {
        console.log(e)
        this.current = e ? 1 : 0;
        // this.state = !e ? 1 : 0;
      },


      goLink(arg) {
        console.log(this.userInfo)
        if (arg == 'start') {
          this.$router.push({
            name: 'Answer',
            params: {
              // user_id: 111111,
              user_id: this.user_id,
              user_name: this.user_name
            }
            // query: {
            //   user_id: this.user_id,
            //   user_name: this.userInfo.NAME,
            //   integral: this.userInfo.INTEGRAL
            // }
          });

        }
        else {
          this.$router.push({path: '/pdf'});
        }
      }
    }
  }

</script>

<style scoped lang="less">
  @import '../../variable';
  //function

  /* .myGrid {
     display: grid;
     grid-template-columns: 98px 148px 313px 138px;
     grid-template-rows: 68px;
     grid-gap: 1px;
   }

   .myGridItem {
     box-sizing: border-box;
     line-height: 68px;
     font-size: 28px;
     text-align: center;
   }*/

  //style
  .main {
    width: 100vw;
    height: @max;
    .main-content {
      /*padding-top: 88px;*/
      box-sizing: border-box;
      height: @max;
      background: linear-gradient(@white, #e5e5e5);
      position: relative;

      .action {
        padding: 32px 24px 96px;
        box-sizing: border-box;
        .user {
          font-size: 27px;
          /*line-height: 70px;*/
          color: #626262;
          flex-grow: 2;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          .name {
            line-height: 46px;
          }
          .time {
            font-size: 34px;
            line-height: 46px;
          }
        }
        .tabber {
          .tab-item {
            width: 141px;
            height: 70px;
            line-height: 70px;
            text-align: center;
            background-color: @white;
            border: 1px solid @theme; /*no*/
            color: #626262;
            flex-grow: 1;
            box-sizing: border-box;
          }
          .tab-item.active {
            background-color: @theme;
            color: @white;
          }
          .tab-item:first-child {
            border-top-left-radius: 5px; /*no*/
            border-bottom-left-radius: 5px; /*no*/
          }
          .tab-item:last-child {
            border-top-right-radius: 5px; /*no*/
            border-bottom-right-radius: 5px; /*no*/
          }
        }
      }

      .content {
        height: 717px;
        padding: 0 24px;
        box-sizing: border-box;
        .tab-ctx {
          /*.bg {
            width: @max;

            .bg-pos {
              width: 630px;
              height: 630px;
              background: url(../../assets/score.png) no-repeat 100%;
              background-size: 100%;
              margin: 0 auto;
              text-align: center;

              span {
                line-height: 630px;
                font-size: 150px;
                color: @white;
              }
            }
          }*/
          .bg {
            width: @max;

            .bg-pos {
              width: 630px;
              height: 630px;
              margin: 0 auto;
              text-align: center;
              h2 {
                padding-top: 139px;
                padding-bottom: 56px;
                font-size: 78px;
                color: #8b8b8b;
              }
              h1 {
                font-size: 156px;
                color: #8b8b8b;
                font-weight: 300;
                font-family: "Times New Roman";

              }
              img {
                width: 336px;
                height: 174px;
              }
            }
          }
          .wrapper-header {
            .grid-header {
              color: @white;
              border: 2px solid #dcdcdc;
              // background: linear-gradient(#f99f9f, #f34141);
              background-color: @theme;
              height: 68px;
              line-height: 68px;
              flex-grow: 1;
              text-align: center;
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              box-sizing: border-box;
            }
            .grid-header:first-child {
              width: 78px;
              border-top-left-radius: 3px; /*no*/
            }
            .grid-header:nth-child(2) {
              width: 176.5px;
            }
            .grid-header:nth-child(3) {
              width: 246.5px;
            }
            .grid-header:nth-child(4) {
              width: 118px;
            }
            .grid-header:last-child {
              width: 78px;
              border-top-right-radius: 3px; /*no*/
            }
          }
          /*.wrapper-header {
            .myGrid();
            .grid-header:first-child {
              border-top-left-radius: 3px; !*no*!
            }
            .grid-header {
              .myGridItem();
              color: @white;
              border: 2px solid @theme; !*no*!
              border-bottom: 0;
              background: linear-gradient(#f99f9f, #f34141);
            }
            .grid-header:last-child {
              border-top-right-radius: 3px; !*no*!
            }
          }*/
          .wrapper-content {
            .grid-item {
              flex-grow: 1;
              background: linear-gradient(#f9f9f9, #e6e6e6);
              border: 1px solid #c9c9c9;
              color: #626262;
              height: 68px;
              line-height: 68px;
              text-align: center;
              overflow: hidden;
              white-space: nowrap;
              text-overflow: ellipsis;
              box-sizing: border-box;
            }
            .grid-item:first-child {
              width: 78px;
            }
            .grid-item:nth-child(2) {
              width: 176.5px;
            }
            .grid-item:nth-child(3) {
              width: 246.5px;
            }
            .grid-item:nth-child(4) {
              width: 118px;
            }
            .grid-item:last-child {
              width: 78px;
            }
          }
          /*.wrapper-content {
            .myGrid();
            .grid-item {
              .myGridItem();
              background: linear-gradient(#f9f9f9, #e6e6e6);
              border: 1px solid #c9c9c9; !*no*!
              color: #626262;
            }
          }*/
          .wrapper-content.active {
            .grid-item {
              color: @theme;
            }
          }
        }

      }
      .scroller-con {
        position: relative;
        width: @max;
      }
      .btn-group {
        /* position: absolute;
         bottom: 0;*/
        width: @max;
        height: 226px;
        text-align: center;
        background-color: #e5e5e5;
        .btn {
          width: 702px;
          padding: 0;
          border: 0;
          font-size: 30px;
          height: 88px;
          line-height: 88px;
          box-sizing: border-box;
          border-radius: 5px; /*no*/
        }
        .start {
          color: @white;
          background-color: @theme;
          margin-bottom: 32px;
        }
        .read {
          color: #636363;
          background-color: @white;
          border: 1px solid #c9c9c9; /*no*/

        }
      }

    }
  }
</style>
