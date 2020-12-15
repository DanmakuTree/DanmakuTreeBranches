<template>
  <div class="container" :style="background">
    <!-- <textarea
      class="input"
      ref="textarea"
      placeholder="弹幕内容"
      v-model="input"
      v-on:keyup='checkInput'
      v-on:keydown='handleEnter'></textarea> -->
    <div class="action">
      <div class="close-button"><a-icon type="close" v-on:click="_close" /></div>
      <div class="other">
        {{ log }}
      </div>
      <div class="move-button"><i class="fas fa-arrows-alt"></i></div>
    </div>
    <transition-group name="list" tag="ul">
      <dmkli v-for="message in messages" :key="message">
        {{message}}
      </dmkli>
    </transition-group>
  </div>
</template>

<script>
  import dmkli from './dmkli.vue'
  import { debounce } from 'lodash'
  import mikejson from './mike.json'
  const supportPlatform = ['BiliBili']
  export default {
    components: {
      dmkli
    },
    data: function () {
      return {
        log: '未设置房间',
        normal: '',
        mainRoom: {
          platform: '',
          roomId: ''
        },
        input: '',
        debounceLog: null,
        debounceMove: null,
        debounceFlex: null,
        debounceSetHeight: null,
        debounceWarnInDanmu: null,
        moving: false,
        followerLog: null,
        giftMap: {},
        messages: [],
        online: 0,
        fans: null,
        vWindow: {
          width: 320,
          height: 24,
          vHeight: 24,
          lowY: null,
          x: null
        },
        mikelist: mikejson.mikelist,
        mikeword: mikejson.mikeword,
        isDev: null
      }
    },
    danmaku: {
      'CurrentWindow.move' () {
        console.log('window moved!')
        this.moving = true
        if (this.debounceWarnInDanmu) {
          this.debounceWarnInDanmu('窗口发生了移动')
        }

        if (this.debounceMove) {
          this.debounceMove()
        }
      },
      'Platform.BiliBili.Service.DanmakuService.Message' (msg) {
        // console.log(msg)
        // var node = document.createElement('li')
        // var list = document.querySelector('#content')
        var text = ''
        var that = this

        var e = msg
        var d = msg.data

        function num2rstr (num) {
          var s = String(num)
          return s.split('').reverse().join('')
        }
        function mikeExists (num) {
          return (that.mikelist.indexOf(num2rstr(num)) !== -1)
        }
        function mikewordExists (comment) {
          return (that.mikeword.indexOf(comment) !== -1)
        }

        if (!d.data.isLotteryAutoMsg && d.type === 'message') {
          text = `%username=${d.data.user.username}% : ${d.data.comment}`
          var medal = `[${d.data.user.medal.label} ${d.data.user.medal.level}] `
          if (medal !== '[ 0] ') { text = medal + text }
          if (mikeExists(d.data.user.uid)) { text = '' }
          if (mikewordExists(d.data.comment)) { text = '' }
        }
        if (d.type === 'block') {
          text = `在房间 ${e.roomId} 内, 用户 %username=${d.data.user.username}% 已被管理员禁言`
        }
        if (d.type === 'welcomeRoomVIP') {
          text = `%warning=前方舰长% %username=${d.data.user.name}% %warning=鲨过来了！%`
        }
        if (d.type === 'superchat') {
          if (d.data.user.medal) {
            medal = `[${d.data.user.medal.label} ${d.data.user.medal.level}] `
          }
          text = `%username=${d.data.user.username}% : ${d.data.comment}`
          if (medal && (medal !== '[ 0] ')) { text = medal + text }
          text = '%warning=[收到了一条SC]%' + text
        }
        if (d.type === 'guardbuy') {
          text = `%username=${d.data.user.username}% %warning=| 购买了舰长%`
        }
        if (d.type === 'gift' && d.data.gift.coinType === 'gold') { // testing with 'silver' | todo: modify to 'gold'
          var vText = `%username=${d.data.user.username}% %warning=投喂了: ${d.data.gift.giftName}%`

          if (!that.giftMap[vText]) {
            that.giftMap[vText] = true
            text = vText

            setTimeout(function () {
              delete that.giftMap[vText]
            }, 1000 * 10) // 10 seconds
          }
        }
        if (d.type === 'live') {
          text = `%warning=所在房间 ${e.roomId} 开播啦！%`
        }
        if (d.type === 'online') {
          that.online = d.data.online
        }

        // node.innerText = text
        // list.appendChild(node)
        // var that = this
        // setTimeout(function () { node.outerHTML = ''; that._flex() }, 20000)
        if (text.length > 0) {
          // this.messages.push(text)
          this.messages.unshift(text)
          setTimeout(function () {
            // that.messages.shift()
            that.messages.pop()
            // that.window.vHeight -= 28
            setTimeout(() => { that._SetHeight() }, 700)
          }, 500 * 1000)
          this.vWindow.vHeight += 28
          this._SetHeight()
          setTimeout(() => { that._SetHeight() }, 900)
          setTimeout(() => { that._SetHeight() }, 1200)
        }
      }
    },
    computed: {
      background () {
        return {
          'background-color': this.dModuleConfig.backgroundColor || 'rgba(0,0,0,0.42)'
        }
      }
    },
    mounted () {
      if (this.dConfigDataLoad) {
        this.dConfigDataLoad.then(() => {
          console.log(this.dModuleConfig)
          if (typeof this.dModuleConfig.windowX === 'number' && typeof this.dModuleConfig.windowY === 'number') {
            this.$currentWindow.setBounds({ x: this.dModuleConfig.windowX, y: this.dModuleConfig.windowY, width: this.vWindow.width, height: this.vWindow.height })
          }
        })
      }

      this.debounceLog = debounce(this._displayLog.bind(this), 3000)
      this.followerLog = debounce(this._displayLog.bind(this), 10000)
      this.debounceMove = debounce(this._move.bind(this), 500)
      this.debounceFlex = debounce(this._flex.bind(this), 50)
      this.debounceSetHeight = debounce(this._SetHeight.bind(this), 50)
      this.debounceWarnInDanmu = debounce(this._warnInDanmu.bind(this), 300)
      this.getMainRoom()
      this.getFollower()
      this.getMikeJsonUpdate()
      // This line will force to set top the window. Use with care. // setInterval(() => { console.log('set top'); this.$currentWindow.moveTop() }, 1000)
      this.vWindow.lowY = this.dModuleConfig.windowY // - this.vWindow.height
    },
    methods: {
      getMainRoom () {
        this.$main.getRoomList().then((roomList) => {
          if (roomList.length > 0) {
            this.mainRoom = roomList[0]
            this.normal = `当前房间: ${this.mainRoom.platform} ${this.mainRoom.roomId}`
            if (supportPlatform.indexOf(this.mainRoom.platform) !== -1) {

            } else {
              this.normal = this.normal + ' 暂不支持的平台'
            }
            this.log = this.normal
          }
        })
      },
      getFollower () {
        var that = this
        setInterval(function () {
          that.$platform.BiliBili.API.getRoomInfo(that.mainRoom.roomId).then((e) => {
            that.fans = e.data.attention
          }).catch(() => {})
        }, 15 * 60000)
        setInterval(function () {
          if (that.fans) { that.displayFollower(`关注人数: ${that.fans}`) }
          setTimeout(() => { that.displayFollower(`房间人气: ${that.online}`) }, 10000)
          setTimeout(() => { that.displayFollower('[辅助视图]弹幕侧边栏') }, 20000)
        }, 30000)
        setInterval(function () {
          if (!that.moving) { that.debounceFlex() }
        }, 800)
      },
      getMikeJsonUpdate () {
        var that = this
        var _interval, _addr
        this.$main.isDev().then((res) => {
          that.isDev = res
          _interval = res ? 30 * 1000 : 3 * 60000
          _addr = res ? 'https://cdn.jsdelivr.net/gh/ax4/dmv-isDev/mike.json' : 'https://raw.githubusercontent.com/DanmakuTree/DanmakuTreeBranches/master/DanmakuTree.DanmuViewer-2077/mike.json'
          setInterval(function () {
            window.fetch(_addr).then(res => res.json())
              .then((_mikejson) => {
                that.mikelist = _mikejson.mikelist
                that.mikeword = _mikejson.mikeword
                console.log(_mikejson)
              })
              .catch(console.warn)
          }, _interval)
        })
      },
      displayLog (text) {
        this.log = text
        this.debounceLog()
      },
      displayFollower (text) {
        this.log = text
        this.followerLog()
      },
      _displayLog () {
        this.log = this.normal
      },
      _move () {
        var newpos = this.$currentWindow.getBounds()
        this.dModuleConfig.windowX = newpos.x
        this.dModuleConfig.windowY = newpos.y
        this.vWindow.lowY = this.dModuleConfig.windowY // + this.vWindow.vHeight
        this.vWindow.x = this.dModuleConfig.windowX
      },
      _flex () {
        // var newpos = this.$currentWindow.getBounds()
        var contentHeight = document.querySelector('.container').offsetHeight
        var positive = contentHeight - this.vWindow.vHeight
        // var diff = (positive > 0) ? Math.floor((positive) / 28) : Math.ceil((positive) / 28)
        var diff = Math.round((positive) / 28)
        this.vWindow.vHeight += diff * 28
        // NOTE: this line makes window not moved
        this.vWindow.lowY = this.dModuleConfig.windowY // + 24 + Math.round((contentHeight - 24) / 28) * 28
        this._SetHeight()
      },
      _SetHeight (delta) {
        var that = this
        if (this.vWindow.vHeight < 560) {
          window.API.CurrentWindow.setBounds({ height: this.vWindow.vHeight, y: that.moving ? undefined : this.vWindow.lowY, width: this.vWindow.width, x: that.moving ? undefined : this.vWindow.x })
        } else {
          window.API.CurrentWindow.setBounds({ height: 584, y: that.moving ? undefined : this.vWindow.lowY, width: this.vWindow.width, x: that.moving ? undefined : this.vWindow.x })
        }
        // console.log(this.$currentWindow.getBounds().y)
        // console.log('lowY', this.vWindow.lowY)
        // console.log('windowY', this.dModuleConfig.windowY)
        // console.log('this.vWindow', this.vWindow)
      },
      _close () {
        this.$currentWindow.close()
      },
      _warnInDanmu (str) {
        // this.messages.push(`%warning=${str}% `)
        var that = this
        if (this.debounceLog) {
          this.log = str
          this.debounceLog()
          console.log('window moved warning set')
        }
        setTimeout(function () {
          that.moving = false
        }, 5000)
      }
    }
  }
  // document.body.onclick = function () {
  //   this._flex()
  // }
  // console.log(window.API.CurrentWindow)
</script>

<style>
html{
  height: fit-content;
}
body{
  background-color: transparent;
  height: fit-content;
  overflow-x: hidden;
}
.container{
  font-size: 18px;
  width: 100%;
  height: fit-content;
  color: white;
  overflow: hidden;
}
.input{
  height: 56px;
  width: 100%;
  resize: none;
  background: initial;
  border: initial;
  padding: 8px;
  outline: initial;
  margin: 0;
}
.action{
  display: flex;
  height: 24px;
}
.action > .move-button{
  width: 1.5rem;
  height: 1.5rem;
  text-align: center;
  -webkit-app-region: drag;
}
.action > .other {
  text-align: center;
  flex-grow: 1;
  -webkit-app-region: drag;
  color: rgb(234,114,147)
}
.username {
  color: #4FC1E9
}
.warning {
  color: #FFC107
}
.list-item {
  display: inline-block;
  line-height: 28px;
  width: 320px;
}
.list-enter-active /*, .list-leave-active */ {
  transition: all 1.2s;
}
.list-enter /*, .list-leave-to /* .list-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(28px);
}
.list-leave-active {
  transition: all 0.8s;
}
.list-leave-to {
  opacity: 0;
  transform: translateY(-28px);
}
/* ::-webkit-scrollbar {         display: none;       } */
</style>
