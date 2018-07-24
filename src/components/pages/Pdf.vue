<template>
  <div>
    <ul>
      <li @click="download('0')">
        <em></em>
        <div>
          <h1>单项选择题.pdf</h1>
          <p>PDF 2,521KB </p>
        </div>

      </li>
      <li @click="download('1')">
        <em></em>
        <div>
          <h1>多项选择题.pdf</h1>
          <p>PDF 1,478KB </p>
        </div>

      </li>
      <li @click="download('2')">
        <em></em>
        <div>
          <h1>判断题.pdf</h1>
          <p>PDF 1,041KB </p>
        </div>

      </li>
      <li @click="download('3')">
        <em></em>
        <div>
          <h1>填空题.pdf</h1>
          <p>PDF 1,056KB </p>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: "Pdf",
    data() {
      return {
        download(val){
          if(val=='0'){
            window.location.href=`http://172.80.72.17:8088/?furl=${config.productUrls}/JSSCZTOnlineAnswer/test/danxuan.pdf`;
          }
          else if(val=='1'){
            window.location.href=`http://172.80.72.17:8088/?furl=${config.productUrls}/JSSCZTOnlineAnswer/test/duoxuan.pdf`;
          }
          else if(val=='2'){
            window.location.href=`http://172.80.72.17:8088/?furl=${config.productUrls}/JSSCZTOnlineAnswer/test/panduan.pdf`;
          }
          else if(val=='3'){
            window.location.href=`http://172.80.72.17:8088/?furl=${config.productUrls}/JSSCZTOnlineAnswer/test/tiankong.pdf`;
          }
        }
      }
    },
    methods:{

    }
  }
</script>

<style scoped lang="less">
  @import '../../variable';

  ul {
    width: @max;
    padding: 0 28px;
    box-sizing: border-box;
    li {
      width: @max;
      height: 134px;
      border-bottom:1px solid #f2f2f2;/*no*/
      box-sizing: border-box;

      em{
        display: inline-block;
        vertical-align: middle;
        background:url(../../assets/pdf.png) no-repeat 0 center;
        background-size:76px 94px;
        width: 76px;
        height: 100%;
        margin-right: 29px;
      }
      div{
        display: inline-block;
        vertical-align: middle;
        h1{
          line-height: 46px;
          font-size: 30px;
          color:#222225;
        }
        p{
          line-height: 32px;
        }
      }

    }

  }

</style>
