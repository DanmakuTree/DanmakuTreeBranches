<template>
  <div class="container">
    <a-card size="small" title="简单弹幕视图重置" style="width: 300px">
      <a slot="extra" href="#">more</a>
      <a-alert v-if="!this.isInstalled('84f22a8f-61a2-4ee2-aba2-db964ed86a4f')" message="该模块未安装" banner />
      <a-alert v-if="this.isInstalled('84f22a8f-61a2-4ee2-aba2-db964ed86a4f')" message="模块已安装" type="success" />
      <p v-if="this.isInstalled('84f22a8f-61a2-4ee2-aba2-db964ed86a4f')" >按下面的按钮对视图进行重置</p>
      <a-button v-if="this.isInstalled('84f22a8f-61a2-4ee2-aba2-db964ed86a4f')" @click="resetModule('84f22a8f-61a2-4ee2-aba2-db964ed86a4f')" type="danger" block>
        重置视图窗口
      </a-button>
    </a-card>
    <br />
    <a-card size="small" title="[辅助]侧边弹幕栏重置" style="width: 300px">
      <a slot="extra" href="#">more</a>
      <a-alert v-if="!this.isInstalled('d4977bd5-ca1a-492d-ac58-2eeb90afa188')" message="该模块未安装" banner />
      <a-alert v-if="this.isInstalled('d4977bd5-ca1a-492d-ac58-2eeb90afa188')" message="模块已安装" type="success" />
      <p>card content</p>
      <p>card content</p>
      <p>card content</p>
    </a-card>
  </div>
</template>

<script>
  export default {
    data: function () {
      return {
        moduleId: '22133289-a1f7-4961-b0bb-7814714ceda0',
        installedModuleList: []
      }
    },
    mounted () {
      this.loadInstallModuleList()
    },
    methods: {
      async resetModule (moduleId) {
        if (!this.isInstalled(moduleId)) { return false }
        var id = moduleId
        var b = await window.API.Module.getModuleWindows(id)
        var wins = Array.isArray(b.data) ? b.data : []
        var defaultWin = (wins.length !== 0) ? wins[0] : false
        console.log(wins, defaultWin)
        await window.API.Module.closeModuleWindows(id, defaultWin)
        await window.API.Module.forcecloseModuleWindows(id, defaultWin)
        await window.API.Module.clearModuleConfig(id)
        await window.API.Module.createModuleExternalWindow(id)
      },
      isInstalled (moduleId) {
        return this.installedModuleList.indexOf(moduleId) !== -1
      },
      loadInstallModuleList () {
        return this.$module.getInstalledModuleList().then((data) => {
          this.installedModuleList = data
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
