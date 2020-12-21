<template>
  <div>
    <p>
      弹幕播报
      <a-switch
        v-model="msgCheck"
        @change="onChange"
        checked-children="播报弹幕"
        un-checked-children="关闭播报" />
      <button v-on:click="offCheck" style="color:white;background-color:red">关闭全部播报</button>
      <button v-on:click="defaultCheck" style="color:white;background-color:green">默认</button>
    </p>
    <p>
      舰长进场播报
      <a-switch
        v-model="guardCheck"
        @change="onChange"
        checked-children="欢迎舰长"
        un-checked-children="关闭播报" />
    </p>
    <p>
      礼物答谢播报
      <a-switch
        v-model="giftCheck"
        @change="onChange"
        checked-children="答谢礼物"
        un-checked-children="关闭播报" />
    </p>
    <p>
      醒目留言播报
      <a-switch
        v-model="superchatCheck"
        @change="onChange"
        checked-children="播报醒目留言"
        un-checked-children="关闭播报" />
    </p>
    <p>
      <button v-on:click="testVoice">点我测试语音姬</button>
      <button v-on:click="cancelVoice" style="color:white;background-color:orange">终止语音</button>
    </p>
    <p id="nightSwitch">
      夜间模式 (1am-7am)
      <a-switch
        v-model="nightCheck"
        :disabled="nightDisable"
        @change="onChange"
        checked-children="夜间低扰"
        un-checked-children="正常播报" />
    </p>
    <p>目前系统上可以使用的语音包有</p>
    <ul>
      <li v-for="voice in voicesTTS" :key="voice.name">
        {{ voice.name }}
      </li>
    </ul>
  </div>
</template>
<script>
  import mikejson from './mike.json'
  export default {
    data: function () {
      return {
        voicesTTS: null,
        msgCheck: false,
        guardCheck: true,
        superchatCheck: true,
        giftCheck: true,
        mikelist: mikejson.mikelist,
        mikeword: mikejson.mikeword,
        nightCheck: false,
        nightDisable: false,
        giftMap: {},
        guardHourMap: {},
        giftHourMap: {}
      }
    },
    danmaku: {
      'Platform.BiliBili.Service.DanmakuService.Message' (msg) {
        var text = ''
        var that = this

        var e = msg
        var d = e.data

        function mikeExists (num) {
          return (that.mikelist.indexOf(num) !== -1)
        }

        function nightMode () {
          var today = new Date()
          var hour = today.getHours()
          var inDuration = (hour >= 1) && (hour <= 7)
          return inDuration && that.nightCheck
        }

        // todo: message should be cancel, with a turn off switch
        if (!d.data.isLotteryAutoMsg && d.type === 'message') {
          text = `${d.data.comment}`
          if (mikeExists(d.data.user.uid)) { return false }
          // if (nightMode()) {text='晚上好'}
          if (that.msgCheck) { that.speak(text) }
        }
        if (d.type === 'welcomeRoomVIP') {
          text = `前方舰长${d.data.user.name}鲨过来了！`
          if (mikeExists(d.data.user.uid)) { return false }
          if (nightMode()) { text = `欢迎${d.data.user.name}` }
          if (that.guardHour(d.data.user.uid) === 'block') { return false }
          if (that.guardCheck) { that.speak(text) }
        }
        if (d.type === 'superchat') {
          text = `收到${d.data.user.username}的Super Chat说：${d.data.comment}`
          if (mikeExists(d.data.user.uid)) { return false }
          if (nightMode()) { text = `谢谢Super Chat: ${d.data.comment}` }
          if (that.superchatCheck) { that.speak(text) }
        }
        if (d.type === 'gift' && d.data.gift.coinType === 'gold') {
          // text = `感谢${d.data.user.username}投喂的${d.data.gift.giftName}，啾咪`
          if (mikeExists(d.data.user.uid)) { return false }
          // if (nightMode()) {text='谢谢礼物'}
          // if (that.giftCheck) { that.speak(text) }
          if (that.giftCheck) { that.thanksGift(d.data.user.uid, d.data.user.username, d.data.gift.giftName, nightMode()) }
        }
      }
    },
    computed: {
    },
    mounted () {
      var that = this
      this.speak('')
      setTimeout(() => { that.getVoices() }, 1000)
      // window.API.CurrentWindow.setTitle('弹幕语音姬')
      this.setTitle('弹幕语音姬')
    },
    methods: {
      speak (msg, lang = 'zh-CN', voice, rate) {
        var synth = window.speechSynthesis
        const u = new SpeechSynthesisUtterance()
        u.lang = lang
        u.text = msg
        u.pitch = 2
        u.rate = 2
        if (voice) { u.voice = voice }
        if (rate) { u.rate = rate }
        synth.speak(u)
      },
      getVoices () {
        var synth = window.speechSynthesis
        var voices = synth.getVoices().sort(function (a, b) {
          const aname = a.name.toUpperCase(); const bname = b.name.toUpperCase()
          if (aname < bname) return -1
          else if (aname === bname) return 0
          else return +1
        })
        console.log('Show and list the voices on your system:', voices)
        this.voicesTTS = voices
      },
      onChange () {
        this.nightDisable = !this.msgCheck && !this.guardCheck && !this.superchatCheck && !this.giftCheck
        console.log(`
msg: ${this.msgCheck}
guard: ${this.guardCheck}
superchat: ${this.superchatCheck}
gift: ${this.giftCheck}
            `)
      },
      testVoice () {
        this.getVoices()
        this.speak('你好我是憨憨弹幕姬。你可以选择你系统上适用的发音包，我现在会的语言有')
        this.speak('English', 'en-US')
        this.speak('普通话', 'zh-CN')
        this.speak('粤语', 'zh-HK')
        this.speak('日本語', 'ja-JP')
        this.speak('한국의', 'ko-KR')
        // this.speak(`我不会唱歌，但我会念
        // 收到username的sc：text
        // 感谢username投喂的gifts，啾咪
        // 前方舰长username鲨过来了！
        // `)
        // this.speak('目前我的语言模板技能是恶魔Ki丽叩撒嘛教会我的，很感谢她！')
        // this.speak('Ki丽叩!Ki丽叩!发出了单推的叫声')
        this.speak('我正在学习其他的技能和多语言切换，希望我早点升级我自己')
      },
      cancelVoice () {
        var synth = window.speechSynthesis
        synth.cancel()
      },
      offCheck () {
        this.cancelVoice()
        this.msgCheck = false
        this.guardCheck = false
        this.superchatCheck = false
        this.giftCheck = false
        this.nightDisable = true
      },
      defaultCheck () {
        this.msgCheck = false
        this.guardCheck = true
        this.superchatCheck = true
        this.giftCheck = true
        this.nightDisable = false
      },
      thanksGift (uid, username, giftName, nightMode) {
        var that = this
        if (!this.giftMap[uid]) {
          that.giftMap[uid] = []
          setTimeout(() => {
            // Joseph's answer, reference: https://stackoverflow.com/questions/15069587/is-there-a-way-to-join-the-elements-in-an-js-array-but-let-the-last-separator-b
            var text = `感谢${username}投喂的${that.giftMap[uid].join(', ').replace(/, ([^,]*)$/, ' 和 $1')}，啾咪`
            if (nightMode) { text = `谢谢${username}的${that.giftMap[uid].join(', ').replace(/, ([^,]*)$/, ' 和 $1')}` }
            if (that.giftHour(uid) > 3) {
              text = `谢谢${that.giftMap[uid].join(', ').replace(/, ([^,]*)$/, ' 和 $1')}，啾咪`
            }
            that.speak(text)
            delete that.giftMap[uid]
          }, 15000)
        }
        if (that.giftMap[uid].includes(giftName) === false) { that.giftMap[uid].push(giftName) }
      },
      guardHour (uid) {
        var that = this
        if (!this.guardHourMap[uid]) {
          that.guardHourMap[uid] = true
          setTimeout(() => {
            delete that.guardHourMap[uid]
          }, 60 * 60000)
          return false
        } else {
          return 'block'
        }
      },
      giftHour (uid) {
        var that = this
        if (!this.giftHourMap[uid]) {
          that.giftHourMap[uid] = 1
          setTimeout(() => {
            delete that.giftHourMap[uid]
          }, 20 * 60000)
          return 1
        } else {
          var prevCount = that.giftHourMap[uid]
          var currCount = prevCount + 1
          that.giftHourMap[uid] = currCount
          return currCount
        }
      },
      setTitle (str) {
        this.$currentWindow.setTitle(str)
      }
    }
  }
</script>

<style scoped>
    #nightSwitch .ant-switch-checked{
        background-color: blueviolet;
    }
</style>
