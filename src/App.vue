<template>
  <div id="app">
    <div class="text-left">当前时间</div>
    <div class="text-left">{{time_now.format('YYYY-MM-DD HH:mm:ss')}}</div>
    <div class="text-left">提交的时间</div>
    <input v-model="timeSel" type="datetime-local">
    <div class="text-left">姓名</div>
    <input v-model="name" type="text" placeholder="姓名">
    <div class="text-left">手机</div>
    <input v-model="mobile" type="text" placeholder="手机号">
    <button @tap="startHandleClick" type="button" class="mui-btn mui-btn-blue">开始</button>
  </div>
</template>

<script>
  let moment = require('moment');
  window.moment = moment
  export default {
    name: 'app',
    components: {},
    data() {
      return {
        moment,
        name: '测试',
        mobile: '111222333',
        // timeSel: moment().format('YYYY-MM-DDTHH:mm'),
        timeSel: moment().format('YYYY-MM-DDT08:59:59'),
        time_wucha: '',//本地时间和北京时间 快 毫秒
        time_now: moment(),//当前时间 动态变化 到点停止
      }
    },
    mounted() {
      window._self = this
      console.log('页面加载完成', moment().format('YYYY-MM-DD HH:mm:ss'))

    },
    methods: {
      startHandleClick() {
        let timer = setInterval(() => {
          this.time_now = moment()
          if (this.time_now.diff(moment(this.timeSel)) > 0) {
            //时间到了
            console.log('时间到了', this.time_now.format('YYYY-MM-DD HH:mm:ss'))
            this.updateStart()
            clearInterval(timer)
          } else {
            console.log(`当前时间: ${this.time_now.format('YYYY-MM-DD HH:mm:ss')}`)
          }
        }, 500)
      },
      //
      updateStart() {
        console.log('开始提交')
        let timer = setInterval(() => {
          this.formAction()
        }, 500)
        setTimeout(() => {
          clearInterval(timer)
        }, 1000)
      },
      computeTimeDiff() {
        //计算时间误差
        this.getBJTime().then(date => {
          let time_bg = moment(date)
          let time_local = moment()
          this.time_wucha = time_local.diff(time_bg)
          console.log('北京时间', time_bg.format('YYYY-MM-DD HH:mm:ss'))
          console.log(`本地时间比北京时间快 ${this.time_wucha} 毫秒`)
        })
      },
      getBJTime() {
        return new Promise((resolve, reject) => {
          $.get('http://api.m.taobao.com/rest/api3.do?api=mtop.common.getTimestamp', data => {
            // console.log(data)
            const date = data.data.t
            resolve(Number(date))
          })
        })
      },
      formAction() {
        this.testUpdate(this.name, this.mobile)
          .then(data => {
            if (data.r === 0) {
              console.log('提交成功', moment().format('YYYY-MM-DD HH:mm:ss'))
            } else {
              console.log('提交失败', moment().format('YYYY-MM-DD HH:mm:ss'), data.r)
            }
          })
          .catch(() => {
            console.log('请求失败')
          })
        // this.xiangQiUpdate('哈哈', '16543253567')
      },
      testUpdate(name, mobile) {
        return new Promise((resolve, reject) => {
          var url = 'http://ka7js75mkc9i2gnz.mikecrm.com/handler/web/form_runtime/handleSubmit.php'
          var params = {
            "cvs": {
              "i": 200103402,
              "t": "OaFtcNB",
              "s": 200237531,
              "acc": "dXnOIBv7wKo4Xh45lfFM28jysYrAgbn4",
              "r": "",
              "c": {
                "cp": {
                  "200992865": name,
                  "200992866": mobile
                }
              }
            }
          }
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
      xiangQiUpdate(name, mobile) {
        return new Promise((resolve, reject) => {
          var url = 'http://gis1z4xshb2s37ki.mikecrm.com/handler/web/form_runtime/handleSubmit.php'
          var params = {
            "cvs": {
              "i": 200241815,
              "t": "dcSGLtc",
              "s": 200710795,
              "acc": "YGtEHjWUXSBV2nlMggbzeklRaAB5MCGD",
              "r": "",
              "c": {
                "cp": {
                  "202474905": name,
                  "202474906": mobile
                },
                "ext": {}
              }
            }
          }
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
    .text-left{
      text-align: left;
    }
  }

</style>
