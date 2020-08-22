<template>
  <div class="container" :style="background">
    <textarea
      class="input"
      ref="textarea"
      placeholder="弹幕内容"
      v-model="input"
      v-on:keyup='checkInput'
      v-on:keydown='handleEnter'></textarea>
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
        debounceMove: null
      }
    },
    danmaku: {
      'CurrentWindow.move' () {
        if (this.debounceMove) {
          this.debounceMove()
        }
      }
    },
    computed: {
      background () {
        return {
          'background-color': this.dModuleConfig.backgroundColor || 'rgba(0,0,0,1)'
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
      /**
       * @param {KeyboardEvent} event
       */
      checkInput (event) {
        if (this.$refs.textarea.value.length <= 0) { return }
        if (event.key !== 'Enter') {
          this.displayLog(`长度: ${this.$refs.textarea.value.length}`)
        }
      },
      handleEnter (event) {
        if (event.key !== 'Enter') {
        } else {
          event.returnValue = false
          this.sendDanmaku()
        }
      },
      sendDanmaku () {
        switch (this.mainRoom.platform) {
        case 'BiliBili':
          this.$platform.BiliBili.API.sendRoomMessage(this.mainRoom.roomId, this.input).then((res) => {
            if (res.code === 0 && res.message === '') {
              this.displayLog('弹幕发送成功')
              this.input = ''
            } else if (res.code === 0) {
              this.displayLog(`弹幕可能失败,Msg: ${res.message}`)
              this.input = ''
            } else {
              this.displayLog(`弹幕发送失败,Code:${res.code},Msg:${res.message}`)
            }
          }).catch((res) => {
            this.displayLog('弹幕发送失败，' + res.error)
          })
          break

        default:
          this.displayLog('暂不支持的平台')
          break
        }
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
      _close () {
        this.$currentWindow.close()
      }
    }
  }
</script>

<style scoped>
body{
  background-color: transparent;
}
.container{
  font-size: 13px;
  width: 100%;
  height: 80px;
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
.action > .close-button{
  width: 1.5rem;
  height: 1.5rem;
  text-align: center;
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
