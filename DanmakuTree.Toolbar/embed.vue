<template>
  <div class="container">
    <a-button @click="openWindow">1234</a-button>
  </div>
</template>

<script>

  export default {
    data: function () {
      return {
        moduleId: 'ce68d480-a08b-4678-a547-65a510083410'
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
      }
    }
  }
</script>

<style>
.container {
  padding: 8px;
}
</style>
