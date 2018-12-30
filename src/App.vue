<template>
  <div id="app">
    <div class="xq-logo-wrap">
      <img src="https://s1.ax1x.com/2018/12/30/Ff5YND.png" alt="">
    </div>
    <div class="form">
      <div ref="formTips" class="form-tips">
        <div class="xq-title">享骑表单自动提交</div>
        <div class="xq-time-tips">本地时间 比 北京时间快约 {{time_wucha}} 毫秒</div>
        <div class="xq-tips">请修改本地时间，尽量控制在1000毫秒以内</div>
      </div>
      <div class="label">姓名</div>
      <cube-input
        placeholder="输入姓名"
        v-model="name"
      ></cube-input>
      <div class="label">手机号</div>
      <cube-input
        v-model="mobile"
        placeholder="输入手机号"
      ></cube-input>
    </div>
    <div class="btn-wrap">
      <cube-button :disabled="btnDisabled" :primary="true" @click="startBtnHandleClick">开始</cube-button>
    </div>
    <div ref="xqPopur" class="xq-popur">
      <div class="xq-popur-bg">
        <div class="now-time">
          <div>{{time_now.format('YYYY-MM-DD')}}</div>
          <div>{{time_now.format('HH:mm:ss')}}</div>
        </div>

        <div class="text-tip">距离{{timeSel.format('MM月DD日')}}9点整</div>
        <div class="seconds-tip">{{parseInt(-countdown/1000)}}秒</div>
        <div class="form-content">
          <div>姓名: {{name}}</div>
          <div>手机: {{mobile}}</div>
        </div>
        <div class="message-tip">数据将在9点整连续提交多次</div>
      </div>

      <div class="log-wrap" ref="logWrap">
        <div :key="i" v-for="(item,i) in logInfoArr_res">{{item}}</div>
      </div>
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
        timeSel: moment(moment().format('YYYY-MM-DDT08:59:59')),
        time_wucha: '',//本地时间和北京时间 快 毫秒
        time_now: moment(),//当前时间 动态变化 到点停止
        logInfoArr: [],
        scrollView: null,
        countdown: 0,//倒计时
        btnDisabled: true,
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
      startBtnHandleClick() {
        if (!(this.name && this.mobile)) {
          this.$createDialog({
            type: 'alert',
            title: '提示',
            content: '请输入姓名和手机号',
            icon: 'cubeic-alert'
          }).show()
          return
        }
        if (moment().diff(this.timeSel) > 0) {
          this.timeSel = this.timeSel.add(1, 'days');
        }
        $(this.$refs.xqPopur).addClass('xq-active')
        this.startHandleClick()
      },
      startHandleClick() {
        let timer = setInterval(() => {
          this.time_now = moment()
          this.countdown = this.time_now.diff(this.timeSel)
          if (this.countdown > 0) {
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
            this.$createDialog({
              type: 'alert',
              title: '提示',
              content: '表单提交结束，期待好消息吧',
            }).show()
          }, 1000)
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
        }).catch(err => {
          this.$createDialog({
            type: 'alert',
            title: '提示',
            content: err,
          }).show()
          const tips = `
          当前浏览器不支持跨域，请在PC端使用chrome内核的浏览器，安装跨域插件后使用。
          下载地址：<a href="http://chromecj.com/web-development/2018-07/1481.html">谷歌跨域扩展插件:Allow-Control-Allow-Origin<a>
          `
          $(this.$refs.formTips).html(tips)
        })
      },
      getBJTime() {
        return new Promise((resolve, reject) => {
          $.ajax({
            url: 'http://api.m.taobao.com/rest/api3.do?api=mtop.common.getTimestamp',
            type: 'get',
            dataType: 'json',
            success(data) {
              const date = data.data.t
              resolve(Number(date))
            },
            error() {
              reject('当前浏览器不支持跨域，请在PC端使用chrome内核的浏览器，安装跨域插件后使用')
            }
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
      },
      testMKForm() {
        this.logPrint(`${moment().format('HH:mm:ss')} 检测表单页面`)
        this.logPrint(`${MK_FORM_URL}`)
        ajaxAction()
          .then(data => {
            if (data === '') {
              this.logPrint(`${moment().format('HH:mm:ss')} 你的IP可能被限制访问`)
              this.$createDialog({
                type: 'alert',
                title: '提示',
                content: '你的IP可能被限制访问',
                icon: 'cubeic-alert'
              }).show()
              return
            }
            if (data.indexOf('SOUL') === -1) {
              this.logPrint(`${moment().format('HH:mm:ss')} 你要提交的表单已经失效`)
              this.$createDialog({
                type: 'alert',
                title: '提示',
                content: '你要提交的表单已经失效',
                icon: 'cubeic-alert'
              }).show()
              return
            }
            this.logPrint(`${moment().format('HH:mm:ss')} 检测表单数据正常`)
            this.btnDisabled = false
          })
          .catch(err => {
            this.$createDialog({
              type: 'alert',
              title: '提示',
              content: err,
              icon: 'cubeic-alert'
            }).show()
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
  body {
    text-align: center;
  }

  #app {
    position: relative;
    display: inline-block;
    width: 350px;
    height: 99vh;
    text-align: initial;
    overflow: hidden;

    .xq-logo-wrap {
      img {
        width: 100%;
      }
    }

    .btn-wrap {
      padding: 30px 20px;

    }

    .form {
      padding: 0 20px;

      .xq-title {
        font-size: 25px;
        text-align: center;
        line-height: 25px;
        margin: 10px 0;
      }

      .xq-time-tips {
        font-size: 14px;
        padding: 2px 0;
        text-align: center;
        color: gray;
      }

      .xq-tips {
        font-size: 14px;
        padding: 2px 0;
        text-align: center;
        color: gray;
      }

      .label {
        height: 40px;
        line-height: 40px;
        margin-top: 15px;
      }

    }
  }

  #app .xq-popur {
    width: 100%;
    height: 100vh;
    background-color: rgba(111, 111, 111, .3);
    background-color: #ffffff;
    position: absolute;
    top: 99vh;
    left: 0;
    transition: top .3s ease;

    .xq-popur-bg {
      background-color: #4cd964;
      padding: 15px 0;

    }

    .now-time {
      font-size: 30px;
      line-height: 40px;
      text-align: center;
    }

    .text-tip {
      font-size: 30px;
      line-height: 40px;
      text-align: center;
      margin: 10px 0;
    }

    .seconds-tip {
      font-size: 50px;
      text-align: center;
      margin: 10px 0;
    }

    .form-content {
      font-size: 20px;
      line-height: 25px;
      text-align: center;
      margin: 10px 0;
    }

    .message-tip {
      font-size: 20px;
      text-align: center;
      margin: 10px 0;
    }

    .log-wrap {
      position: absolute;
      height: calc(99vh - 365px);
      width: 100%;
      left: 0;
      bottom: 30px;
      padding: 10px;
      background-color: rgba(0, 0, 0, .2);
      overflow-y: scroll;

      & > div {
        width: calc(100% - 20px);
        display: inline-block;
        font-size: 18px;
        margin: 10px 0;
        overflow: hidden;
      }
    }

    &.xq-active {
      top: 0;
    }
  }

</style>
