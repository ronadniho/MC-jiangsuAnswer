<template>
  <div>
    <!--<Header-->
      <!--:title="title"-->
      <!--:href="href"-->
    <!--/>-->
    <!--看题-->
    <div class="container answer" v-if="layerToggle&&!message">
      <!--页码-->
      <div class="action flex">
        <div class="tip">
          <span>{{page}}</span>
          <span>/{{total}}</span>
        </div>
        <div class="action-btn flex flex-align-center">
          <a @click="answerCardToggle">
            <img src="../../assets/answer.png" alt="">
            <span>题库</span>
          </a>

          <!--<a @click="signToggle" v-for="(item,index) in signList" v-if="index+1==page">-->
            <!--<img src="../../assets/sign.png" alt="">-->
            <!--<span>{{item.sign?'已标记':'标记'}}</span>-->
          <!--</a>-->
        </div>
      </div>

      <!--主内容-->
      <div class="content">
        <!--标题-->
        <div class="subject">
          <b>{{examName}}</b>
        </div>
        <!--答案-->
        <div class="result">
          <!--item-->
          <div class="result-item" v-for="(item,index) in examlist" :key="index">
            <div v-if="index+1==(page%basic==0?basic:page%basic)" style="width:100%;">
              <!--题目-->

              <div class="title">
                {{item.QUESTIONS_CONTENT}}
              </div>
              <!--item-->
              <div class="box-item single" v-if="item.QUESTIONS_TYPE==0">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx">
                  <input type="radio"
                         :id="'radio'+item.ID+idx"
                         :value="child"
                         v-model="item.reference_answer"
                         disabled
                  />
                  <label :for="'radio'+item.ID+idx">{{child}}</label>
                </div>


              </div><!--单选-->

              <div class="box-item multi" v-if="item.QUESTIONS_TYPE==1">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx">
                  <input type="checkbox"
                         :id="'checkbox'+item.ID+idx"
                         :value="child"
                         v-model="item.reference_answer"
                         disabled
                  />
                  <label :for="'checkbox'+item.ID+idx">{{child}}</label>
                </div>
              </div><!--多选-->

              <div class="box-item single" v-if="item.QUESTIONS_TYPE==2">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx">
                  <input type="radio"
                         :id="'radio'+item.ID+idx"
                         :value="child"
                         v-model="item.reference_answer"
                         disabled
                  />
                  <label :for="'radio'+item.ID+idx">{{child}}</label>
                </div>

              </div><!--判断-->

              <div class="tip">

                <!--单选题或判断题提示-->
                <div class="spans" v-if="item.QUESTIONS_TYPE!=1">
                  <div @click="show(item)" class="flex flex-align-center">
                    <span></span>
                    <em>查看提示</em>
                  </div>

                  <b v-if="item.status">{{item.PROMPT}}</b>
                </div>


                <!--多选提示-->
                <div class="spans" v-if="item.QUESTIONS_TYPE==1" >
                  <div @click="show(item)" class="flex flex-align-center">
                    <span></span>
                    <em>查看提示</em>
                  </div>

                  <b v-if="item.status">{{item.PROMPT}}</b>
                </div>

              </div>

            </div>
          </div>
        </div>

      </div>

      <!--控制器-->
      <div class="submit flex flex-justify-center">
        <div>
          <button :class="page>1?'':'no'" @click="prev">上一题</button>
        </div>
        <div>
          <button :class="page<total?'':'no'" @click="next">下一题</button>
        </div>
        <!--<div v-if="page==total">
          <button @click="submit">交卷</button>
        </div>-->
      </div>
    </div>

    <!--答题卡-->
    <div class="container answerCard" v-if="!layerToggle&&!message">
      <div class="subject flex">
        <b>
          {{examName}}
        </b>
        <a @click="answerCardToggle">&times;</a>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>单选题</dt>
          <dd class="lt" v-for="(item,index) in signList" v-if="item.type=='single'">
            <span @click="guide(index)"
                  :data-index="index+1"
                  :class="[item.sign?'sign':'']"
            >{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>多选题</dt>
          <dd class="lt" v-for="(item,index) in signList" v-if="item.type=='multiple'">
            <span @click="guide(index)"
                  :data-index="index+1"
                  :class="[item.sign?'sign':'']"
            >{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>判断题</dt>
          <dd class="lt" v-for="(item,index) in signList" v-if="item.type=='judge'">
            <span @click="guide(index)"
                  :data-index="index+1"
                  :class="[item.sign?'sign':'']"
            >{{index+1}}</span>
          </dd>
        </dl>
      </div>

    </div>

    <!--loading-->
    <Loading v-if="loading"/>

    <!--warning-->
    <!--<div class="modal warning" v-if="message">
      <div class="modal-diag">
        <div class="modal-bg">
          <p>你有一次考试没提交，是否退出考试?</p>
        </div>
        <div class="btn-group">
          <button @click="close">退出</button>
          <button @click="back">取消</button>
        </div>
      </div>
    </div>-->

  </div>
</template>
<script>
  import Header from '../common/Header';
  import Loading from '../common/Loading';
  import {request} from '../../api/common'
  import {times} from '../../api/time'
  import browser from '../../api/checkVersion';
  import $ from 'jquery';


  export default {
    name: 'ReadAnswer',
    components: {
      Header,
      Loading,
    },
    data() {
      return {
        title: '题库',
        href: -1,
        layerToggle: true,//true为默认答题状态
        total: 0,//总题数
        signList: [],
        totalPage: 0,//总页数
        index: 0,
        basic: 30,//basic和PAGECOUNT初始化必须同步
        PAGECOUNT: 30,
        page: 1,//当前页码
        user_id: '',
        user_name: '',
        studentId: '',
        sessionId: '',
        examlist: [],//题目集合
        examName: '财政厅岗位技能竞赛',//标题
        exampass: 0,//通过分数线
        loading: true,//加载动画
        result: false,//得分页状态
        message: '',
        endTime: '',
        startTime: '',
        state: false,
        version: '',
      }
    },
    // beforeRouteEnter(to, from, next) {
    //   console.log(to)
    //   var userInfo = sessionStorage.getItem("userInfo");
    //   next(vm => {
    //     if (!userInfo) {
    //       vm.$router.push({path: "login"});
    //     } else {
    //       next();
    //     }
    //   });
    // },
    created: function () {
      console.log(this.global.isLogin)
      this.getClassExamlist();
    },
    methods: {
      show(val){
        console.log(val);
        val.status = !val.status;
      },
      getClassExamlist() {
        request.getServerData(
          {
            'PAGESTART': this.index * this.basic,
            'PAGECOUNT': this.PAGECOUNT,
          },
          "onlineExamination.action.examinationAction[getAllQuestions]",
          true,
          (result) => {
            this.loading = true;
            console.log(result)
            console.log(result.examinationPaperList)

            /*var signList = JSON.parse(localStorage.getItem('signList'));
            if (signList) {
              this.signList = signList;
              this.total = signList.length;
            }
            else {
              var signList = [];//标记
              var singleList = [];//单选
              var multipleList = [];//多选
              var judgeList = [];//判断
              for (var i = 0; i < result.singleSelectioncnt.singleSelectioncnt; i++) {
                singleList[i] = {
                  type: 'single',
                  sign: false,
                };
                signList.push(singleList[i])
              }
              console.log(singleList)
              for (var i = 0; i < result.multipleSelectioncnt.multipleSelectioncnt; i++) {
                multipleList[i] = {
                  type: 'multiple',
                  sign: false,
                };
                signList.push(multipleList[i])
              }
              for (var i = 0; i < result.judgeSelectioncnt.judgeSelectioncnt; i++) {
                judgeList[i] = {
                  type: 'judge',
                  sign: false,
                };
                signList.push(judgeList[i])
              }
              this.signList = signList;
              this.total = parseInt(result.singleSelectioncnt.singleSelectioncnt) + parseInt(result.multipleSelectioncnt.multipleSelectioncnt) + parseInt(result.judgeSelectioncnt.judgeSelectioncnt);
              localStorage.setItem('signList', JSON.stringify(signList));
            }*/

            var signList = [];//标记
            var singleList = [];//单选
            var multipleList = [];//多选
            var judgeList = [];//判断
            for (var i = 0; i < result.singleSelectioncnt.singleSelectioncnt; i++) {
              singleList[i] = {
                type: 'single',
                sign: false,
              };
              signList.push(singleList[i])
            }
            console.log(singleList)
            for (var i = 0; i < result.multipleSelectioncnt.multipleSelectioncnt; i++) {
              multipleList[i] = {
                type: 'multiple',
                sign: false,
              };
              signList.push(multipleList[i])
            }
            for (var i = 0; i < result.judgeSelectioncnt.judgeSelectioncnt; i++) {
              judgeList[i] = {
                type: 'judge',
                sign: false,
              };
              signList.push(judgeList[i])
            }
            this.signList = signList;
            this.total = parseInt(result.singleSelectioncnt.singleSelectioncnt) + parseInt(result.multipleSelectioncnt.multipleSelectioncnt) + parseInt(result.judgeSelectioncnt.judgeSelectioncnt);

            if (this.index) {
              console.log(this.index * this.basic);
             result.examinationPaperList.map(val=>{
                val.status=false;
              })
              this.examlist = result.examinationPaperList;
              this.loading = false;
            }
            else {
              result.examinationPaperList.map(val=>{
                val.status=false;
              })
              this.examlist = result.examinationPaperList;
              this.loading = false;
            }
          });
      },
      common(data) {
        this.examlist = data;
        this.total = data.length;

        this.loading = false;
        console.log(data)
      },
      guide(e) {
        this.loading = true;
        console.log(e)
        this.layerToggle = !this.layerToggle;

        this.page = e + 1;
        console.log(parseInt(this.page / this.basic));
        this.index = parseInt(this.page / this.basic);
        this.getClassExamlist();
      },
      answerCardToggle() {
        this.layerToggle = !this.layerToggle;
      },
      close() {
        this.$router.go(-1);
      },
      back() {
        this.message = false;
      },
      /*signToggle() {
        console.log(this.page)
        console.log(this.signList)
        this.signList[this.page - 1].sign = !this.signList[this.page - 1].sign;
        localStorage.setItem('signList', JSON.stringify(this.signList));
      },*/
      next() {
        if (this.page < this.total) {
          this.page++;
          if (Number.isInteger((this.page - 1) / this.basic) && (this.index + 1 == (this.page - 1) / this.basic)) {
            this.loading = true;
            this.index++;
            console.log(this.index)
            this.getClassExamlist();
          }
        }
      },
      prev() {
        if (this.page > 1) {
          this.page--;
          console.log(Number.isInteger((this.page) / this.basic))
          if (Number.isInteger(this.page / this.basic) && (this.index == this.page / this.basic)) {
            this.loading = true;
            this.index--;
            this.getClassExamlist();
          }
        }
      },
    }
  }
</script>

<style scoped lang="less">
  @import '../../variable';

  .container {
    width: @max;
    height: @max;
    /*padding-top: 88px;*/
    box-sizing: border-box;
  }

  .answer {
    padding-bottom: 152px;
    .action {
      padding: 0 24px;
      box-sizing: border-box;
      border-bottom: 1px solid @answer-col-c9; /*no*/
      .tip {
        flex-grow: 1;
        .base(@answer-col-62, @answer-col-62);
        line-height: 88px;
        span {
          .base(@theme, @answer-font-32);
        }
        span + span {
          .base(@answer-col-62, @answer-font-24);
        }
      }
      .action-btn {
        flex-grow: 1;
        justify-content: flex-end;
        a {
          width: 138px;
          height: 58px;
          border: 1px solid @answer-col-c9; /*no*/
          border-radius: 3px; /*no*/
          padding-left: 22px;
          margin-left: 10px;
          img {
            width: 26px;
            height: 33px;
            vertical-align: middle;
          }
          span {
            padding-left: 10px;
            vertical-align: middle;
            line-height: 58px;
            .base(@answer-col-62, @answer-font-20);
          }
        }
      }
    }

    .content {
      padding: 0 24px;
      width: @max;
      box-sizing: border-box;
      .subject {
        b {
          .base(@answer-col-62, @answer-font-32);
          line-height: 88px;
          width: 728px;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
      .result {
        width: @max;
        padding-top: 16px;
        box-sizing: border-box;
        .result-item{
          width: @max;
          >div{
            width:@max;
          }
        }
        .title {
          .base(@answer-col-62, @answer-font-32);
          line-height: 56px;
        }
        .single {
          .box {
            width: 100%;
            min-height: 69px;
            padding: 15px 0;
          }

          input {
            display: none;
          }

          label {
            width: 100%;
            height: 100%;
            display: inline-block;
            position: relative;
            line-height: 69px;
            .base(@answer-col-62, @answer-font-32);
            padding-left: 100px;
            box-sizing: border-box;
          }
          label:active {
            background: #EEEEEE;
          }
          label:after {
            content: ""; /*必须设置*/
            display: inline-block;
            width: 70px;
            height: 70px;
            border: 1px solid @answer-col-c9; /*no*/
            position: absolute;
            top: 0;
            left: 15px;
            border-radius: 50%;
            box-sizing: border-box;
          }

          input:checked + label:after {
            /*background-color: @theme;*/
            background: url(../../assets/raido.png) no-repeat;
            background-size: cover;
          }
        }
        .multi {
          .box {
            width: 100%;
            min-height: 70px;
            padding: 15px 0;
          }
          input {
            display: none;
          }
          label {
            width: 100%;
            height: 100%;
            display: inline-block;
            line-height: 75px;
            position: relative;
            .base(@answer-col-62, @answer-font-32);
            padding-left: 100px;
            box-sizing: border-box;
          }
          label:active {
            background: #EEEEEE;
          }
          label:after {
            content: "";
            width: 71px;
            height: 71px;
            /*border: 1px solid @answer-col-c9; !*no*!*/
            display: inline-block;
            position: absolute;
            top: 0;
            left: 15px;
            background: url(../../assets/no-multi.png) no-repeat;
            background-size: 100%;
          }
          input:checked + label:after {
            content: "";
            /* color: @white;
             text-align: center;
             font-size: 60px;*/
            background: url(../../assets/multi.png) no-repeat;
            background-size: 100%;
          }

        }
        .tip {
          width: @max;
          .spans {
            font-size: 30px;
            margin-top: 46px;
            width: @max;
            color: @answer-col-62;
            overflow: hidden;
            div{
              padding-bottom: 50px;
            }
            span{
              display: inline-block;
              height: 25px;
              border-left: 10px solid @theme;
            }
            em{
              padding-left: 15px;
              box-sizing: border-box;
              height: 40px;
              /*border-left: 10px solid @theme;*/
              font-style: normal;
            }
            b{
              font-weight:400;
              color:#000;
              height: 300px;
              line-height: 40px;
              word-wrap: break-word;
            }
          }
        }
      }
    }

    .submit {
      position: fixed;
      bottom: 0;
      background-color: @white;
      width: @max;
      height: 152px;
      padding-top: 32px;
      box-sizing: border-box;
      text-align: center;
      div {
        flex-grow: 1;
        button {
          background-color: @theme;
          border-radius: 5px; /*no*/
          border: 0;
          width: 258px;
          line-height: 88px;
          .base(@white, @answer-font-30);
        }
        button.no {
          color: @answer-col-62;
          background-color: @white;
          border: 1px solid @answer-col-c9; /*no*/
        }
      }

    }
  }

  .answerCard {
    padding-bottom: 152px;
    .subject {
      padding: 0 27px;
      box-sizing: border-box;
      border-bottom: 1px solid @answer-col-c9; /*no*/
      b {
        .base(@answer-col-62, @answer-font-32);
        line-height: 88px;
        width: 728px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      a {
        display: inline-block;
        font-size: 50px;
        height: 50px;
        line-height: 88px;
      }
    }
    .guide {
      padding-top: 36px;
      dl {
        width: @max;
        dt {
          .base(@answer-col-62, @answer-font-28);
          padding: 0 24px;
          box-sizing: border-box;

        }
        dd {
          .base(@answer-col-62, @answer-font-28);
          width: 20%;
          height: 108px;
          display: flex;
          justify-content: center;
          align-items: center;
          span {
            display: block;
            border-radius: 50%;
            border: 4px solid @answer-col-c9; /*no*/
            .base(@answer-col-62, @answer-font-28);
            width: 68px;
            height: 68px;
            line-height: 68px;
            text-align: center;
          }
          span.sel {
            background-color: @theme;
            color: @white;
          }
          span.sign {
            border-color: #1F86FF
          }
        }
      }
    }
    .submit {
      position: fixed;
      bottom: 0;
      background-color: @white;
      width: @max;
      height: 152px;
      padding-top: 32px;
      box-sizing: border-box;
      text-align: center;
      button {
        background-color: @theme;
        border-radius: 5px; /*no*/
        border: 0;
        width: 500px;
        line-height: 88px;
        .base(@white, @answer-font-30);
      }

    }
  }

  .modal {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    width: @max;
    height: @max;
    background-color: rgba(0, 0, 0, .4);
    z-index: 1000;
    img {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 300px;
      height: 300px;
      margin-left: -150px;
      margin-top: -150px;
    }
    .modal-diag {
      position: absolute;
      top: 50%;
      left: 50%;
      margin-left: -310px;
      margin-top: -309px;
      .modal-bg {
        width: 620px;
        height: 520px;
        background: url(../../assets/modal-bg.png) no-repeat;
        background-size: cover;
        position: relative;
        p {
          width: 520px;
          color: @theme;
          font-size: 30px;
          position: absolute;
          bottom: 46px;
          left: 50%;
          margin-left: -260px;
          text-align: center;
        }
      }
      .btn-group {
        width: @max;
        height: 98px;
        display: table;
        content: '';
        clear: both;
        button {
          color: @white;
          font-size: 30px;
          background-color: @theme;
          height: 98px;
          line-height: 98px;
          border: 0;
          width: @max/2;
          box-sizing: border-box;
          padding: 0;
          margin: 0;
          float: left;
        }
        button:last-child {
          background-color: #918181;
        }
      }
    }
  }

  /*.modal {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    width: @max;
    height: @max;
    background-color: rgba(0, 0, 0, .4);
    z-index: 1000;
    img {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 300px;
      height: 300px;
      margin-left: -150px;
      margin-top: -150px;
    }
    .modal-diag {
      position: absolute;
      top: 50%;
      left: 50%;
      margin-left: -310px;
      margin-top: -309px;
      .modal-bg {
        width: 620px;
        height: 520px;
        background: url(../../assets/modal-bg.png) no-repeat;
        background-size: 100%;
        position: relative;
        p {
          width: 520px;
          color: @theme;
          font-size: 30px;
          position: absolute;
          bottom: 46px;
          left: 50%;
          margin-left: -260px;
          text-align: center;
        }
      }
      .btn {
        width: @max;
        height: 98px;
        button {
          color: @white;
          font-size: 30px;
          background-color: @theme;
          height: 98px;
          line-height: 98px;
          border: 0;
          width: @max;
          box-sizing: border-box;
          padding: 0;
          margin: 0;
        }
      }
    }
  }*/
</style>
