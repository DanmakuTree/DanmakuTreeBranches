<template>
  <div class="container" :style="background">
    sss
  </div>
</template>

<script>
  export default {
    data: function () {
      return {

      }
    },
    computed: {
      background () {
        return {
          'background-color': this.dModuleConfig.backgroundColor
        }
      }
    },
    mounted () {
      console.log(this)
    },
    methods: {}
  }
</script>

<style>
body{
  background-color: transparent;
}
.container{
  width: 100%;
  height: 100%;
}
</style>
