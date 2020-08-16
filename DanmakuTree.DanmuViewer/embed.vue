<template>
  <div class="container">
    <a-button @click="openWindow">打开窗口</a-button>
    <a-button @click="requestQuickLink">添加快捷方式到快捷栏</a-button>
    <a-button @click="resetWindow">重置窗口</a-button>
    <a-button @click="closeWindow">关闭窗口</a-button>
    背景颜色
    <a-input placeholder="Basic usage" v-model="dModuleConfig.backgroundColor"/>
    {{dModuleConfig.backgroundColor}}
  </div>
</template>

<script>
  export default {
    data: function () {
      return {
        moduleId: '84f22a8f-61a2-4ee2-aba2-db964ed86a4f'
      }
    },
    methods: {
      openWindow () {
        this.$module.createModuleExternalWindow(this.moduleId).then((res) => {
          if (res.code !== 0) {
            this.$message.error(res.msg)
          }
        }).catch((e) => {
          this.$message.error(e.message || e)
          console.error(e)
        })
      },
      requestQuickLink () {
        this.$module.requestQuickLink(this.moduleId, {
          name: '快捷弹幕栏',
          action: 'createModuleExternalWindow',
          data: {}
        })
      },
      resetWindow () {
        // var containerHeight = document.querySelector('.container').offsetHeight
        // this.$currentWindow.setBounds({ x: 0, y: 0, height: containerHeight })
      },
      closeWindow () {
        this.$module.getModuleWindows(this.moduleId).then((e) => {
          if (Array.isArray(e.data)) {
            e.data.forEach(element => {
              this.$module.closeModuleWindows(this.moduleId, element)
            })
          }
        })
      }
    }
  }
</script>

<style>
.container {
  padding: 8px;
}
</style>
