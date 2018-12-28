<template>
  <div id="app">
    <div class="text-left">本地时间 比 北京时间快约 {{time_wucha}} 毫秒</div>
    <div class="text-left">提交的时间</div>
    <input v-model="timeSel" type="datetime-local" readonly="readonly">
    <div class="text-left">姓名</div>
    <input v-model="name" type="text" placeholder="姓名" maxlength="4">
    <div class="text-left">手机</div>
    <input v-model="mobile" type="text" placeholder="手机号" maxlength="11">
    <button ref="submitBtn" @tap="startHandleClick" type="button" class="mui-btn mui-btn-blue">开始</button>
    <div class="log-div text-left">
      <!-- <div class="mui-scroll-wrapper"> -->
      <!-- <div class="mui-scroll"> -->
      <div class="log-wrap" ref="logWrap">
        <div :key="i" v-for="(item,i) in logInfoArr_res">{{item}}</div>
      </div>
      <!-- </div> -->
      <!-- </div> -->
    </div>
  </div>
</template>

<script>
  let moment = require('moment');
  // const ENVIRONMENT = 'TEST'
  const ENVIRONMENT = 'XQ'
  //麦客网地址
  const MK_FORM_URL = 'http://gis1z4xshb2s37ki.mikecrm.com/JI5p0O4'//xq的
  // const MK_FORM_URL = 'http://gis1z4xshb2s37ki.mikecrm.com/dcSGLtc'//失效的
  // const MK_FORM_URL = 'http://ka7js75mkc9i2gnz.mikecrm.com/OaFtcNB'//测试的
  // const MK_FORM_URL = 'http://ka7js75mkc9i2gnz.mikecrm.com/wsEqMGV'//不在时间范围
  /////////////
  const TEST_URL = 'http://ka7js75mkc9i2gnz.mikecrm.com/handler/web/form_runtime/handleSubmit.php'
  //test: {"cvs":{"i":200103402,"t":"OaFtcNB","s":200238984,"acc":"TIXaEDgbjvYpGW0lLaGnmHpnavB798sm","r":"","c":{"cp":{"200992865":"11111","200992866":"11111"}}}}
  const XQ_URL = 'http://gis1z4xshb2s37ki.mikecrm.com/handler/web/form_runtime/handleSubmit.php'
  //12-27: {"cvs":{"i":200241815,"t":"dcSGLtc","s":200710795,"acc":"YGtEHjWUXSBV2nlMggbzeklRaAB5MCGD","r":"","c":{"cp":{"202474905":name,"202474906":mobile},"ext":{}}}}
  //12-28: {"cvs":{"i":200243921,"t":"JI5p0O4","s":200712652,"acc":"1ulXDVmPcXkgiujwkJHY0Ijdrcv5K3q4","r":"","c":{"cp":{"202500664":"周杰","202500665":"17756202920"},"ext":{}}}}
  //
  window.moment = moment
  export default {
    name: 'app',
    components: {},
    data() {
      return {
        moment,
        name: '',
        mobile: '',
        timeSel: moment().format('YYYY-MM-DDT08:59:59'),
        // timeSel: moment().format('YYYY-MM-28T00:58:00'),
        time_wucha: '',//本地时间和北京时间 快 毫秒
        time_now: moment(),//当前时间 动态变化 到点停止
        logInfoArr: [],
        scrollView: null,
      }
    },
    computed: {
      logInfoArr_res() {
        return this.logInfoArr.slice(0, 50)
      }
    },
    mounted() {
      window._self = this
      // this.scrollView = mui('.mui-scroll-wrapper').scroll()
      this.logPrint(`${moment().format('HH:mm:ss')}  页面加载完成`)
      this.computeTimeDiff()
    },
    methods: {
      startHandleClick() {
        this.$refs.submitBtn.disabled = true
        this.$refs.submitBtn.innerHTML = '正在运行...'
        let timer = setInterval(() => {
          this.time_now = moment()
          if (this.time_now.diff(moment(this.timeSel)) > 0) {
            //时间到了
            this.logPrint(`${this.time_now.format('HH:mm:ss')}  时间到了`)
            this.updateStart()
            clearInterval(timer)
          } else {
            this.logPrint(`${this.time_now.format('HH:mm:ss')}  正在等待...`)
          }
        }, 500)
      },
      //
      updateStart() {
        this.logPrint(`${moment().format('HH:mm:ss')}  开始提交`)
        let timer = setInterval(() => {
          this.formAction()
        }, 300)
        setTimeout(() => {
          clearInterval(timer)
          setTimeout(() => {
            this.logPrint(`${moment().format('HH:mm:ss')}  运行结束`)
          }, 1000)
          this.$refs.submitBtn.innerHTML = '运行结束'
        }, 3000)
      },
      computeTimeDiff() {
        //计算时间误差
        this.getBJTime().then(date => {
          let time_bg = moment(date)
          let time_local = moment()
          this.time_wucha = time_local.diff(time_bg)
          this.logPrint(`本地时间, ${time_local.format('YYYY-MM-DD HH:mm:ss')}`)
          this.logPrint(`北京时间, ${time_bg.format('YYYY-MM-DD HH:mm:ss')}`)
          this.logPrint(`本地时间比北京时间快 ${this.time_wucha} 毫秒`)
          this.testMKForm()
        })
      },
      getBJTime() {
        return new Promise((resolve, reject) => {
          $.get('http://api.m.taobao.com/rest/api3.do?api=mtop.common.getTimestamp', data => {
            const date = data.data.t
            resolve(Number(date))
          })
        })
      },
      formAction() {
        this.testUpdate(this.name, this.mobile)
          .then((data) => {
            if (data.r === 0) {
              this.logPrint(`${moment().format('HH:mm:ss')}  提交成功`)
            } else {
              this.logPrint(`${moment().format('HH:mm:ss')}  提交失败  ${data.r}`)
            }
          })
          .catch((data) => {
            this.logPrint(`请求失败`)
          })
      },
      testUpdate(name, mobile) {
        this.logPrint(`${moment().format('HH:mm:ss')} 提交表单`)
        let url = ''
        let params = {}
        if (ENVIRONMENT === 'TEST') {
          url = TEST_URL
          params = {
            "cvs": {
              "i": 200103402,
              "t": "OaFtcNB",
              "s": 200239023,
              "acc": "3tvYej4CTjQJx0k9rZJ678xHQ3ndoi1q",
              "r": "",
              "c": {"cp": {"200992865": name, "200992866": mobile}}
            }
          }
          if (ENVIRONMENT === 'XQ') {
            url = XQ_URL
            params = {
              "cvs": {
                "i": 200243921,
                "t": "JI5p0O4",
                "s": 200712652,
                "acc": "1ulXDVmPcXkgiujwkJHY0Ijdrcv5K3q4",
                "r": "",
                "c": {"cp": {"202500664": name, "202500665": mobile}, "ext": {}}
              }
            }
          }
          return new Promise((resolve, reject) => {
            $.ajax({
              url,
              type: 'post',
              dataType: 'json',
              data: {
                d: JSON.stringify(params)
              },
              success(data) {
                resolve(data);
              },
              error(data) {
                reject(data)
              }
            })
          })
        }
      },
      testMKForm() {
        this.logPrint(`${moment().format('HH:mm:ss')} 检测表单页面`)
        this.logPrint(`${MK_FORM_URL}`)
        ajaxAction()
          .then(data => {
            if (data === '') {
              this.logPrint(`${moment().format('HH:mm:ss')} 你的IP可能被限制访问`)
              mui.alert('你的IP可能被限制访问')
              return
            }
            if (data.indexOf('SOUL') === -1) {
              this.logPrint(`${moment().format('HH:mm:ss')} 你要提交的表单已经失效`)
              mui.alert('你要提交的表单已经失效')
              return
            }
            this.logPrint(`${moment().format('HH:mm:ss')} 检测表单数据正常`)
          })
          .catch(err => {
            mui.alert(err)
          })

        function ajaxAction() {
          return new Promise((resolve, reject) => {
            $.ajax({
              url: MK_FORM_URL,
              success(data) {
                resolve(data)
              },
              error(data) {
                console.log('error', data)
                reject('请求失败')
              }
            })
          })
        }
      },
      logPrint(msg) {
        this.logInfoArr.unshift(msg)
      }
    },
  }
</script>

<style lang="scss" type="text/scss">
  #app {
    position: relative;
    display: inline-block;
    width: 350px;
    margin-left: 50%;
    transform: translateX(-50%);
    text-align: center;

    .mui-btn {
      width: 100%;
      margin-bottom: 15px;
    }

    .text-left {
      text-align: left;
    }

    .log-div {
      position: relative;
      height: 200px;
      background-color: white;
      overflow: hidden;
      overflow-y: scroll;
    }

    .log-wrap > div {
      height: 30px;
      line-height: 30px;
    }
  }

</style>
