<template>
  <div class="container" :style="background">
    <ul id='content'>
      <li v-for="message in messages" :key="message">
        {{message}}
      </li>
    </ul>
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
  </div>
</template>

<script>
  import { debounce } from 'lodash'
  const supportPlatform = ['BiliBili']
  export default {
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
        giftMap: {},
        messages: []
      }
    },
    danmaku: {
      'CurrentWindow.move' () {
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
        if (!d.data.isLotteryAutoMsg && d.type === 'message') {
          text = `${d.data.user.username} : ${d.data.comment}`
          var medal = `[${d.data.user.medal.label} ${d.data.user.medal.level}] `
          if (medal !== '[ 0] ') { text = medal + text }
        }
        if (d.type === 'block') {
          text = `在房间 ${e.roomId} 内, 用户 ${d.data.user.username} 已被管理员禁言`
        }
        if (d.type === 'welcomeRoomVIP') {
          text = `前方舰长 ${d.data.user.name} 鲨过来了！`
        }
        if (d.type === 'superchat') {
          if (d.data.user.medal) {
            medal = `[${d.data.user.medal.label} ${d.data.user.medal.level}] `
          }
          text = `${d.data.user.username} : ${d.data.comment}`
          if (medal !== '[ 0] ') { text = medal + text }
          text = '收到了一条SC，' + text
        }
        if (d.type === 'guardbuy') {
          text = `${d.data.user.username} | 购买了舰长`
        }
        if (d.type === 'gift' && d.data.gift.coinType === 'gold') { // testing with 'silver' | todo: modify to 'gold'
          var vText = `${d.data.user.username} 投喂了: ${d.data.gift.giftName}`

          if (!that.giftMap[vText]) {
            that.giftMap[vText] = true
            text = vText

            setTimeout(function () {
              delete that.giftMap[vText]
            }, 1000 * 10) // 10 seconds
          }
        }
        if (d.type === 'live') {
          text = '所在房间开播啦！'
        }

        // node.innerText = text
        // list.appendChild(node)
        // var that = this
        // setTimeout(function () { node.outerHTML = ''; that._flex() }, 20000)
        if (text.length > 0) {
          this.messages.push(text)
          setTimeout(function () { that.messages.splice(0, 1); that.debounceFlex() }, 20000)
          this.debounceFlex()
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
            this.$currentWindow.setBounds({ x: this.dModuleConfig.windowX, y: this.dModuleConfig.windowY })
          }
        })
      }

      this.debounceLog = debounce(this._displayLog.bind(this), 3000)
      this.debounceMove = debounce(this._move.bind(this), 500)
      this.debounceFlex = debounce(this._flex.bind(this), 100)
      this.getMainRoom()
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
      displayLog (text) {
        this.log = text
        this.debounceLog()
      },
      _displayLog () {
        this.log = this.normal
      },
      _move () {
        var newpos = this.$currentWindow.getBounds()
        this.dModuleConfig.windowX = newpos.x
        this.dModuleConfig.windowY = newpos.y
      },
      _flex () {
        var newpos = this.$currentWindow.getBounds()
        this.$currentWindow.setBounds({ height: document.querySelector('.container').offsetHeight, y: newpos.y - document.querySelector('.container').offsetHeight + newpos.height })
      },
      _close () {
        this.$currentWindow.close()
      }
    }
  }
  document.body.onclick = function () {
    this._flex()
  }
</script>

<style>
html{
  height: fit-content;
}
body{
  background-color: transparent;
  height: fit-content;
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
}
</style>
