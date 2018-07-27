<template>
  <div :class="[prizeStatus?'add':'']">
    <!--<Header-->
    <!--:title="title"-->
    <!--:href="href"-->
    <!--:jumpState="jumpState"-->
    <!--:alert="alert"-->
    <!--@child="warning"-->
    <!--/>-->
    <!--答题-->
    <div class="container answer" v-if="layerToggle&&!message&&!prizeStatus">
      <!--页码-->
      <div class="action flex">
        <div class="tip">
          <span>{{page}}</span>
          <span>/{{totalPage}}</span>
        </div>
        <div class="action-btn flex flex-align-center">
          <a @click="answerCardToggle">
            <img src="../../assets/answer.png" alt="">
            <span>答题卡</span>
          </a>
          <!--<a @click="signToggle" v-for="(item,index) in examlist" v-if="index+1==page">
            <img src="../../assets/sign.png" alt="">
            <span>{{item.sign?'已标记':'标记'}}</span>
          </a>-->
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
          <div class="result-item" v-for="(item,index) in examlist" :key="item.ID">
            <div v-if="index+1==page" style="width:100%;">
              <!--题目-->
              <div class="title">
                {{item.QUESTIONS_CONTENT}}
              </div>
              <!--item-->
              <div class="box-item single" v-if="item.QUESTIONS_TYPE==0">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx"
                >
                  <input type="radio"
                         :id="'radio'+item.ID+idx"
                         :value="child"
                         v-model="item.ANSWER"
                         :disabled="item.is_true=='0'"
                  />
                  <label :for="'radio'+item.ID+idx">{{child}}</label>
                </div>

              </div><!--单选-->

              <div class="box-item multi" v-if="item.QUESTIONS_TYPE==1">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx"
                >
                  <input type="checkbox"
                         :id="'checkbox'+item.ID+idx"
                         :value="child"
                         v-model="item.ANSWER"
                         :disabled="item.is_true=='0'"
                  />
                  <label :for="'checkbox'+item.ID+idx">{{child}}</label>
                </div>
              </div><!--多选-->

              <div class="box-item single" v-if="item.QUESTIONS_TYPE==2">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx"
                >
                  <input type="radio"
                         :id="'radio'+item.ID+idx"
                         :value="child"
                         v-model="item.ANSWER"
                         :disabled="item.is_true=='0'"
                  />
                  <label :for="'radio'+item.ID+idx">{{child}}</label>
                </div>

              </div><!--判断题-->

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
                <div class="spans" v-if="item.QUESTIONS_TYPE==1">
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
        <div v-if="page<=totalPage">
          <!--<button :class="page==totalPage?'no':''" @click="next" :disabled="btnDisabled">下一题</button>-->
          <!--<button v-if="page!==totalPage" @click="next" :disabled="btnDisabled">下一题</button>-->
          <button v-if="page!=totalPage" @click="next">下一题</button>
          <button v-if="page==totalPage" @click="end">完成</button>
        </div>
        <!--<div v-if="page==totalPage">-->
        <!--<button @click="submit">交卷</button>-->
        <!--</div>-->
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

          <dd class="lt" v-for="(item,index) in examlist" v-if="item.QUESTIONS_TYPE==0">
            <span @click="guide(index)" :data-index="index+1"
                  :class="[(item.is_true!='1')?'sel':'', item.sign?'sign':'']">{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>多选题</dt>
          <dd class="lt" v-for="(item,index) in examlist" v-if="item.QUESTIONS_TYPE==1">
            <span @click="guide(index)" :data-index="index+1"
                  :class="[(item.is_true!='1')?'sel':'', item.sign?'sign':'']">{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>判断题</dt>
          <dd class="lt" v-for="(item,index) in examlist" v-if="item.QUESTIONS_TYPE==2">
            <span @click="guide(index)" :data-index="index+1"
                  :class="[(item.is_true!='1')?'sel':'', item.sign?'sign':'']">{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <!--<div class="submit">-->
      <!--<button @click="submit">交卷并查看结果</button>-->
      <!--</div>-->
    </div>


    <!--中奖信息-->
    <div id="prize" v-if="prizeStatus">

      <img src="../../assets/yxlm.png" alt="">
      <div class="prize-message">
        <div class="prize-title">奖学金</div>
        <div class="prize-context">
          <div class="mony"><b>小惊喜</b></div>
          <div class="prize-text">
            <p>学霸{{user_name}}</p>
            <p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;恭喜获得奖学金！请于活动结束前到人教处（1017室）登记。</p>
          </div>
        </div>
      </div>
      <div class="btn">
        <button @click="jump">返回首页</button>
      </div>
    </div>


    <!--loading-->
    <Loading v-if="loading"/>
    <div class="wait" v-if="wait"></div>

  </div>
  <!-- <div class="modal" v-if="result" @click="resultToggle">
     <div>
       <h4>{{message}}</h4>
     </div>
   </div>-->
</template>

<script>
  import Header from '../common/Header';
  import Loading from '../common/Loading';
  import {request} from '../../api/common'
  import {times} from '../../api/time'
  import {__GlobalInfo} from "../../api/config";
  import browser from '../../api/checkVersion';

  export default {
    name: 'Answer',
    components: {
      Header,
      Loading,
    },
    data() {
      return {
        title: '考试',
        href: -1,
        jumpState: false,
        // jumpState: true,
        alert: true,
        layerToggle: true,//true为默认答题状态
        totalPage: 0,//总页数
        page: 1,//当前页码
        user_id: '',
        user_name: '',
        integral: '',
        sessionId: '',
        examlist: [],//题目集合
        examName: '财政厅岗位技能竞赛',//标题
        exampass: 0,//通过分数线
        loading: true,//加载动画
        result: false,//得分页状态
        message: '',
        rewardInfo: {},
        endTime: '',
        startTime: '',
        state: false,
        clo: true,
        version: '',
        count: 0,
        btnDisabled: true,//答对当前题目才能进入下一题,false为答对状态
        CREATE_TIME: '',//每天00:00清除缓存
        wait: false,
        prizeStatus: false,
        baseRandoms: [
          '今天的知识点有没有好好消化呢？明天也要活力满满地接受知识哦？',
          '即使学海无涯，也阻挡不了小可爱们一颗爱学习的心，对不对？',
          '好习惯的养成就是要每天坚持，小可爱们加油哦！？',
          '世界上最好的保鲜就是不断进步，让自己成为一个更好和更值得爱的人。',
          '愿以后你的遇见都配得上你的运气，你的得到配得上你的努力。',
          '别忘了经常查看自己在积分榜的排名哦',
          //length=6
        ],
        manyRandoms: [
          '前几天的收获不小吧？重在积累，要坚持下去哦',
          '今天的才华更胜昨天，听说才华和颜值成正比哦！',
          '你这么优秀，我一定要去告诉其他的小伙伴！',
          '今天小伙伴不在线，他们去查字典找词语夸奖你了……',
        ],
        Randoms500: [
          '哇，传说中的天目四大财主•••额，四大才子就是你么？',
          '太赞了，你这是要霸占天目头条的节奏么！',
          '确认过眼神，你就是Top10！'
        ],
      }
    },
    /*  beforeRouteEnter(to, from, next) {
       console.log(to)
       next(vm => {
         if (!vm.global.isLogin) {
           console.log(vm.$router)
           vm.$router.push({path: 'login'})
         }
         else {
           next();

         }
       });
     },
     beforeRouteLeave(to, from, next) {
       console.log(to.path);
       console.log(from.path);
       console.log(this.$router);
       console.log(history);
       console.log('count:'+this.count)
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

       if (to.path == "/main" && this.global.isLogin&&!this.count) {
         console.log(this.global.isLogin);
         // next();
         // this.$router.push({ path: "login" });
         mui.confirm("本次答题没提交是否退出", "", ["是", "否"], alerts, "div");
       } else {
         next();
       }

       // next();
       // 导航离开该组件的对应路由时调用
       // 可以访问组件实例 `this`
     },
   */
    created: function () {
      this.title = times();
      // console.log(this.$route.params)
      this.user_id = this.$route.params.user_id;
      this.user_name = this.$route.params.user_name;
      // console.log('user_id:' + this.user_id)
      // this.getAnswercnt();

      this.getClassExamlist(this.user_id);


    },
    watch: {
      /* examlist: {
         handler(newValue, oldValue) {
           console.log('--------')
           console.log(oldValue)
           console.log(newValue)
           if (!oldValue.length && newValue.length) {//20题
             console.log('第一次进页面')
             return;
           } else {
             console.log(this.examlist[this.page - 1])
             if (this.examlist[this.page - 1].QUESTIONS_TYPE != 1) {
               this.myWatch(this.examlist[this.page - 1], '0')
             }
             else {
               this.myWatch(this.examlist[this.page - 1], '1')
             }
           }

           // for (let i = 0; i < newValue.length; i++) {
           //   if (oldValue[i] != newValue[i]) {
           //     console.log(newValue)
           //   }
           // }
         },
         deep: true
       }*/
    },
    methods: {
      jump() {
        this.$router.go(-1);
      },
      randomToast() {
        console.log(__GlobalInfo.sessionKey.userInfo)
        // let userInfo = JSON.parse(sessionStorage.getItem(__GlobalInfo.sessionKey.userInfo));
        let userInfo = this.getSession(__GlobalInfo.sessionKey.userInfo);
        console.log(userInfo)
        if (userInfo && userInfo.INTEGRAL >= 500 && userInfo.num) {
          var res = this.baseRandoms.concat(this.Randoms500, this.manyRandoms);
          console.log(res)
        }
        else if (userInfo && userInfo.INTEGRAL <= 500 && userInfo.num) {
          var res = this.baseRandoms.concat(this.manyRandoms);
          console.log(res)
        }
        else if (userInfo && userInfo.INTEGRAL >= 500 && !userInfo.num) {
          var res = this.baseRandoms.concat(this.Randoms500);
          console.log(res)
        }
        else {
          var res = this.baseRandoms;
          console.log(res)
        }
        let idx = Math.floor(Math.random() * res.length);
        return '恭喜你，完成今天的学习任务！' + res[idx];
      },
      show(val) {
        console.log(val);
        val.status = !val.status;
      },
      warning() {
        this.$router.go(-1);
        // var self = this;
        // mui.confirm('本次答题没提交是否退出', '退出', ['是', '否'], self.alerts, 'div');
      },
      /*alerts(e) {
        this.count = 1;
        !e.index && this.$router.go(-1);
      },*/
      /*getAnswercnt() {
        var self = this;
        request.getServerData(
          {
            // user_id:this.user_id,
            user_id: '111111'
          },
          "onlineExamination.action.examinationAction[getAnswercnt]",
          true,
          (result) => {
            console.log(result)
            console.log(result.responsesnum.RESPONSESNUM)
            if (result.responsesnum.RESPONSESNUM == '1') {//今天次数用完了
              this.count = 1;
              mui.alert('今日答题次数已用完明天再来', '', '返回', self.$router.go(-1), 'div');
            }
            else {
              this.getClassExamlist(this.user_id);
            }
          });
      },*/
      getClassExamlist(user_id, callback) {
        request.getServerData(
          {
            user_id: user_id,
          },
          "onlineExamination.action.examinationAction[getExaminationPaper]",
          true,
          (result) => {
            console.log(result)
            if (result.rewardInfo) {
              this.rewardInfo = result.rewardInfo;
            }
            if (result.examinationPaperList.length) {
              result.examinationPaperList.map(val => {
                val.status = false;
                this.CREATE_TIME = val.CREATE_TIME;
                val.DISABLED = false;
                if (val.QUESTIONS_TYPE != 1) {/*不是多选题，是单选题或判断题*/
                  if (val.is_true == '0') {
                    val.ANSWER = val.reference_answer;
                    val.DISABLED = true;
                  }
                  else {
                    val.ANSWER = '';
                    val.DISABLED = false;
                  }
                }
                else {/*多选题*/
                  if (val.is_true == '0') {
                    val.ANSWER = val.REFERENCE_ANSWER.split('|');
                    val.reference_answer = val.REFERENCE_ANSWER.split('|');
                    val.DISABLED = true;
                  }
                  else {
                    val.ANSWER = [];
                    val.reference_answer = val.REFERENCE_ANSWER.split('|');
                    val.DISABLED = false;
                  }
                }
                val.sign = false;
              });
              this.examlist = result.examinationPaperList;
              this.totalPage = this.examlist.length;
              this.loading = false;

              console.log(this.examlist)
              this.btnDisabled = !this.examlist[this.page - 1].DISABLED;
              this.setSession(this.examlist);
              if (callback) {
                callback();
              }
              return this.examlist;
            }
            /* this.startTime = times();
             if (result.msg) {/!*次数用完*!/
               this.loading = false;
               this.message = result.msg;
             }
             else {
               if (result.multipleSelectionList.length) {
                 result.multipleSelectionList.map(val => {
                   result.singleSelectionList.push(val)
                 })
               }
               result.singleSelectionList.map(val => {
                 val.startTime = this.startTime;
                 val.QUESTIONS_TYPE == 0 ? (val.ANSWER = '') : (val.ANSWER = []);
                 val.sign = false;
               })
               this.examlist = result.singleSelectionList;
               console.log(this.examlist)
               this.totalPage = this.examlist.length;
               this.loading = false;
             }*/
          });

      },
      myWatch(arg, type) {
        if (arg.is_true != '0') {
          console.log(type)

          console.log(arg.ANSWER)
          switch (type) {
            case '0':
              if (arg.ANSWER == arg.REFERENCE_ANSWER) {
                // console.log('提交后台')
                // arg.is_true = '0';
                // this.btnDisabled = false;
                // this.setSession(this.examlist);
                // this.submit(arg);
              }
              break;

            case '1':

              if (JSON.stringify(Array.prototype.sort.call(arg.ANSWER)) == JSON.stringify(Array.prototype.sort.call(arg.reference_answer))) {
                // console.log('提交后台')
                // arg.is_true = '0';
                // if (this.page != this.totalPage) {
                //   this.btnDisabled = false;
                // }
                // this.setSession(this.examlist);
                // this.submit(arg, 1);
              }
              break;

            case '2':
              if (arg.ANSWER == arg.REFERENCE_ANSWER) {
                // console.log('提交后台')
                // arg.is_true = '0';
                // this.btnDisabled = false;
                // this.setSession(this.examlist);
                // this.submit(arg);
              }
              break;
          }
        }
      },
      common(data) {
        this.examlist = data;
        this.totalPage = data.length;
        // this.examName = data[0].answerList[0].category;

        this.loading = false;
        this.setSession(data);
        console.log(data)
      },
      guide(e) {
        console.log(e)
        this.layerToggle = !this.layerToggle;
        this.page = e + 1;
      },

      setSession(val) {
        sessionStorage.setItem([this.user_id + '_everyday'], JSON.stringify(val));
      },
      getSession(userid) {
        return JSON.parse(sessionStorage.getItem([userid]));
      },
      removeSession(userid) {
        return sessionStorage.removeItem([userid]);
      },
      answerCardToggle() {
        this.layerToggle = !this.layerToggle;
        this.setSession(this.examlist);
      },
      signToggle() {
        console.log(this.page)
        this.examlist[this.page - 1].sign = !this.examlist[this.page - 1].sign;
        this.setSession(this.examlist);
      },
      checkTotalExamIsTrue() {//查找所有题是否都答对
        for (var i = 0, examlist = this.examlist; i < examlist.length; i++) {
          console.log(examlist[i])
          if (examlist[i].QUESTIONS_TYPE == '0' || examlist[i].QUESTIONS_TYPE == '2') {
            if (examlist[i].ANSWER !== examlist[i].reference_answer) {
              break;
            }
          }
          if (examlist[i].QUESTIONS_TYPE == '1') {
            if (Array.prototype.join.call(examlist[i].ANSWER, '|') !== Array.prototype.join.call(examlist[i].reference_answer, '|')) {
              // console.log(Array.prototype.join.call(examlist[i].ANSWER, '|'))
              // console.log(Array.prototype.join.call(examlist[i].reference_answer, '|'))
              // console.log(examlist[i].reference_answer)
              // console.log(examlist[i].ANSWER)
              break;
            }
          }
        }
        console.log('i:' + i)
        console.log('i:' + examlist.length)
        if (i != examlist.length) {
          this.loading = false;
          mui.alert('您还有未答完的题目', '', '返回', null, 'div');
        }
        else {
          this.loading = false;
          mui.alert(this.randomToast(), '', '确定', () => {
            console.log(this.rewardInfo)
            if (this.rewardInfo.ID) {//用户抽过奖了
              //跳回首页
              this.$router.go(-1);
            }
            else {//抽奖
              this.draw();
            }

          }, 'div');
        }
      },
      end() {
        this.loading = true;
        if (this.examlist[this.page - 1].is_true == 0) {//最后一题答对状态不需要再次提交
          this.checkTotalExamIsTrue();
        }
        else {//最后一题没有答对先提交，在判断是否所有题目都答对了
          this.filterSubmit(true);
        }
      },
      draw() {//TODO: 抽奖
        console.log(this.user_id)
        request.getServerData(
          {
            user_id: this.user_id
          },
          "onlineExamination.action.examinationAction[reward]",
          true,
          result => {
            console.log(result);
            if (result.is_reward == '0') {//抽奖结果，0表示中奖，1表示未中奖
              this.prizeStatus = true;
            }
            else {
              this.$router.go(-1);
            }
          }
        )
      },
      submit(arg, type, callback) {
        console.log(arg)
        if (type) {//1为多选题
          if (arg.ANSWER.length > 1) {
            // arg.ANSWER = Array.prototype.join.call(arg.ANSWER, '|');
            var answer = Array.prototype.join.call(arg.ANSWER, '|');
          }
          else if (arg.ANSWER.length == 0) {
            // arg.ANSWER = '';
            var answer = '';
          }
          else {
            // arg.ANSWER = Array.prototype.join.call(arg.ANSWER, '');
            var answer = Array.prototype.join.call(arg.ANSWER, '');
          }
        }
        console.log(arg.ANSWER)
        let requestObj = {
          questions_id: arg.ID,
          pitch_on_option: type?answer:arg.ANSWER,
          answer_results: arg.is_true,
          VALUE: arg.VALUE,
          user_id: this.user_id,
          user_name: this.user_name,
        }
        console.log(requestObj)
        request.getServerData(
          requestObj,
          "onlineExamination.action.examinationAction[markingPapers]",
          true,
          (result) => {
            console.log(result)
            // this.wait = false;
            if (result == '请求失败') {
              mui.alert('服务器出错', '', '返回', self.back, 'div')
            }
            else {
              var res = JSON.stringify(result);
              // if (arg.QUESTIONS_TYPE == '1') {//提交完成将多选题答案切成数组
              //   arg.ANSWER = arg.ANSWER.split('|');
              // }
              if (callback) {
                callback();
              }
              // this.$router.push({
              //   path: 'answerResult',
              //   query: {res}
              // })
            }

            // this.$router.go(-1);
            // var res = result;
            // localStorage.removeItem([this.classid]);
            // res.state = this.state;
            // var resStr = JSON.stringify(res)
            // this.$router.push({
            //   path: 'answerResult',
            //   query: {resStr}
            // });
          })

      },
      next() {
        if (this.page < this.totalPage) {
          this.loading = true;
          console.log(this.examlist[this.page - 1].is_true == '0')
          if (this.examlist[this.page - 1].is_true == '0') {//以答
            this.btnDisabled = false;
            this.page++;
            this.loading = false;
          }
          else {//未答
            console.log(this.examlist[this.page - 1]);
            this.filterSubmit();
          }
          // this.btnDisabled = !this.examlist[this.page - 1].DISABLED;
          this.setSession(this.examlist);

        }
      },
      filterSubmit(arg) {
        if (this.examlist[this.page - 1].QUESTIONS_TYPE != 1) {//单选题或判断题
          if (this.examlist[this.page - 1].ANSWER == this.examlist[this.page - 1].reference_answer) {//选对了
            let callback = () => {
              this.examlist[this.page - 1].is_true = '0';
              if (arg) {
                this.checkTotalExamIsTrue();
              }
              else {
                this.loading = false;
                this.page++;
              }
            }
            this.examlist[this.page - 1].is_true = '0';
            this.submit(this.examlist[this.page - 1], 0, callback);
          }
          else {//选错了
            this.loading = false;
            mui.alert('真遗憾，答错了！没关系，可查看提示后再回答哦，加油！', '', '返回', null, 'div');
            this.btnDisabled = true;
          }
        }
        else {//多选题
          if (JSON.stringify(Array.prototype.sort.call(this.examlist[this.page - 1].ANSWER)) == JSON.stringify(Array.prototype.sort.call(this.examlist[this.page - 1].reference_answer))) {//选对了
            let callback = () => {
              this.examlist[this.page - 1].is_true = '0';
              if (arg) {
                this.checkTotalExamIsTrue();
              }
              else {
                this.loading = false;
                this.page++;
              }
            };
            this.examlist[this.page - 1].is_true = '0';
            this.submit(this.examlist[this.page - 1], 1, callback);
          }
          else {//选错了
            this.loading = false;
            mui.alert('真遗憾，答错了！没关系，可查看提示后再回答哦，加油！', '', '返回', null, 'div');
            this.btnDisabled = true;
          }
        }
      },
      prev() {
        if (this.page > 1) {
          this.page--;
          if (this.examlist[this.page - 1].is_true == '0') {
            this.btnDisabled = false;
          }
          else {
            this.btnDisabled = true;
          }
          // this.btnDisabled = !this.examlist[this.page - 1].DISABLED;
          this.setSession(this.examlist);
        }
      },
    }
  }
</script>

<style scoped lang="less">
  @import '../../variable';

  .add {
    height: 100%;
  }

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
        padding-top: 16px;
        box-sizing: border-box;
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
            /*width: 71px;*/
            width: 72px;
            height: 71px;
            /*border: 1px solid @answer-col-c9; !*no*!*/
            display: inline-block;
            position: absolute;
            top: 0;
            left: 15px;
            background: url(../../assets/no-multi.png) no-repeat;
            background-size: cover;
          }
          input:checked + label:after {
            content: "";
            /* color: @white;
             text-align: center;
             font-size: 60px;*/
            background: url(../../assets/multi.png) no-repeat;
            background-size: cover;
          }

        }
        /* .tip {
           span {
             font-size: 30px;
             margin-top: 46px;
             border-left: 10px solid @theme;
             line-height: 48px;
             height: 48px;
             display: block;
             color: @answer-col-62;
             padding-left: 15px;
             box-sizing: border-box;
             b {
               color: #ff3030;
               word-wrap: break-word
             }
           }
         }*/
        .tip {
          width: @max;
          .spans {
            font-size: 30px;
            margin-top: 15px;
            width: @max;
            color: @answer-col-62;
            overflow: hidden;
            div {
              padding-bottom: 13px;
            }
            span {
              display: inline-block;
              height: 25px;
              border-left: 10px solid @theme;
            }
            em {
              padding-left: 15px;
              box-sizing: border-box;
              height: 40px;
              /* border-left: 10px solid @theme;*/
              font-style: normal;
            }
            b {
              font-weight: 400;
              color: @answer-col-62;
              height: 300px;
              line-height: 50px;
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

  .wait {
    background-color: transparent;
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 2000;
  }

  #prize {
    padding-top: 72px;
    box-sizing: border-box;
    /*position: fixed;*/
    /*left: 0;*/
    /*right: 0;*/
    /*top: 0;*/
    /*bottom: 0;*/

    width: @max;
    height: @max;
    background: url(../../assets/prize.png) no-repeat;
    -webkit-background-size: cover;
    background-size: cover;
    text-align: center;
    img {
      width: 345px;
      height: 406px;
      margin-bottom: 42px;
    }
    .prize-message {
      position: relative;
      width: 484px;
      height: 472px;
      border: 0;
      border-radius: 5px; /*no*/
      background-color: #ffe9b3;
      padding: 7px;
      box-sizing: border-box;
      margin: 0 auto 70px;
      .prize-title {
        position: absolute;
        z-index: 100;
        top: -45px;
        color: #fff1d9;
        font-weight: 700;
        font-size: 48px;
        width: @max;
        height: 73px;
        line-height: 73px;
        background: url(../../assets/title.png) no-repeat center center;
        -webkit-background-size: 374px 73px;
        background-size: 374px 73px;
        text-align: center;
      }
      .prize-context {
        border: 3px dashed #ffd092; /*no*/
        border-radius: 5px; /*no*/
        padding: 55px 18px 0;
        box-sizing: border-box;
        width: @max;
        height: @max;
        .mony {
          background: url(../../assets/mony.png) no-repeat;
          background-size: cover;
          width: 281px;
          height: 180px;
          text-align: center;
          margin: 0 auto;
          b {
            color: @white;
            font-size: 38px;
            line-height: 180px;
          }

        }
        .prize-text {
          p {
            color: #ff690b;
            font-size: 28px;
            line-height: 58px;
            text-align: left;
          }
        }
      }
    }
    .btn {
      button {
        width: 385px;
        height: 82px;
        background-color: #ffa23f;
        border: 0;
        border-radius: 37px;
        color: @white;
        font-size: 36px;
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
