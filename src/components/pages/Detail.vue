<template>
  <div class="content">
    <!--排名-->
    <div class="tab-ctx">
      <div class="wrapper-header flex">
        <div class="grid-header">序号</div>
        <div class="grid-header">姓名</div>
        <div class="grid-header">日期</div>
        <div class="grid-header">积分</div>
      </div>
      <!--<div class="wrapper-content active flex" v-if="userRanking&&my">-->
      <!--<div class="grid-item">{{userRanking.ROWNO}}</div>-->
      <!--<div class="grid-item">{{userRanking.NAME}}</div>-->
      <!--<div class="grid-item">{{userRanking.ACADEMY}}</div>-->
      <!--<div class="grid-item">{{userRanking.INTEGRAL}}</div>-->
      <!--</div>-->
      <div class="scroller-con">
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
            <div class="wrapper-content flex" v-for="(val,index) in integralInfo"
                 :class="[val.hasClass?'active':'']">
              <div class="grid-item">{{index+1}}</div>
              <div class="grid-item">{{val.USER_NAME}}</div>
              <div class="grid-item">{{val.QUESTION_TIME}}</div>
              <div class="grid-item">{{val.DAILYINTEGRAL}}</div>
            </div>

          </div>


        </scroller>

      </div>


    </div>
    <!--&lt;!&ndash;排名&ndash;&gt;
    <div class="tab-ctx">
      <div class="wrapper-header flex">
        <div class="grid-header">序号</div>
        <div class="grid-header">姓名</div>
        <div class="grid-header">日期</div>
        <div class="grid-header">积分</div>
      </div>
      &lt;!&ndash;<div class="wrapper-content active flex" v-if="userRanking&&my">&ndash;&gt;
      &lt;!&ndash;<div class="grid-item">{{userRanking.ROWNO}}</div>&ndash;&gt;
      &lt;!&ndash;<div class="grid-item">{{userRanking.NAME}}</div>&ndash;&gt;
      &lt;!&ndash;<div class="grid-item">{{userRanking.ACADEMY}}</div>&ndash;&gt;
      &lt;!&ndash;<div class="grid-item">{{userRanking.INTEGRAL}}</div>&ndash;&gt;
      &lt;!&ndash;</div>&ndash;&gt;
      <div class="scroller-con">
        <scroller
          :on-refresh="refresh"
          :on-infinite="infinite"
          :noDataText="'没有数据'"
          ref="my_scroller"
        >
          &lt;!&ndash;<div style="height: 1px;"></div>&ndash;&gt;
          &lt;!&ndash; content goes here &ndash;&gt;
          <div id="wrapper-list" class="wrapper-list">

            &lt;!&ndash;item&ndash;&gt;
            <div class="wrapper-content flex" v-for="(val,index) in 20"
                 :class="[val.hasClass?'active':'']">
              <div class="grid-item">1</div>
              <div class="grid-item">1</div>
              <div class="grid-item">1</div>
              <div class="grid-item">1</div>
            </div>

          </div>


        </scroller>

      </div>


    </div>-->
  </div>
</template>

<script>
  import $ from 'jquery'
  import {request} from '../../api/common';

  export default {
    name: "Detail",
    data() {
      return {
        integralInfo: [],//排行集合
        rankingItemH: 68,//每条高度
        PAGESTART: 0,
        PAGECOUNT: 9,//每次加载十条
        h: '',
        my: true,
        loadList: true,
      }
    },
    created: function () {
      console.log(this.$route.params)
      console.log(this.$route.params.user_id)
      this.user_id = this.$route.params.user_id;
      this.PAGESTART = -1;
    },
    mounted: function () {
      // var htmlFS = $('html').css('fontSize').replace('px', '');
      // console.log(htmlFS)
      // this.h = ((this.PAGECOUNT * this.rankingItemH) / 2 / htmlFS) + 'rem';
      // $('.scroller-con').css('height', this.h)
      console.log($('._v-container'))
      $('._v-container').css('paddingTop','33px')
    },
    methods: {
      refresh(done) {
        setTimeout(() => {
          this.PAGESTART = -1;
          this.loadList = true;
          this.integralInfo = [];
          this.postList(done);
        }, 100)
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
          // this.postList(done);
        }, 100)
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
          "onlineExamination.action.examinationAction[getIntegralInfo]",
          true,
          (result) => {
            console.log(result)
            console.log('PAGESTART:' + this.PAGESTART)
            if (result.integralInfo) {
              if (this.PAGESTART) {
                if (result.integralInfo.length) {//如果是数组//一条以上是数组
                  result.integralInfo.forEach((val)=>{
                    this.integralInfo.push(val);
                  })
                }

                else {//否则是对象//一条数据是对象
                  this.integralInfo.push(result.integralInfo);
                }
              }
              else {
                if (result.integralInfo.length) {
                  this.integralInfo = result.integralInfo;
                }
                else{
                  this.integralInfo=[];
                  this.integralInfo.push(result.integralInfo);
                }
              }
              fn();
            }

            else {
              this.loadList = false;
              fn(true);
            }


            // this.loading = false;
          });
      },
    }
  }
</script>

<style scoped lang="less">
  @import '../../variable';

 /* ._v-container {
    top: 33px; !*no*!
  }*/

  .content {
    width: @max;
    height: @max;
    position: absolute;
    top: 0;
    left: 0;
    overflow: hidden;
    .tab-ctx {
      .bg {
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
      }
      .wrapper-header {
        position: relative;
        z-index: 1000;
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
          width: 98px;
          border-top-left-radius: 3px; /*no*/
        }
        .grid-header:nth-child(2) {
          width: 148px;
        }
        .grid-header:nth-child(3) {
          width: 313px;
        }
        .grid-header:last-child {
          width: 138px;
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
          width: 98px;
        }
        .grid-item:nth-child(2) {
          width: 148px;
        }
        .grid-item:nth-child(3) {
          width: 313px;
        }
        .grid-item:last-child {
          width: 138px;
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
</style>
