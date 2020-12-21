<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <HelloWorld/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/map/index.vue';

export default {
  name: 'Home',
  components: {
    HelloWorld
  }
}
</script>
<style lang="less" scoped>
.home {
  margin: 0 auto;
  background: url('../assets/bac.png') 0 0 no-repeat;
  width: 1200px;
  height: 600px;
}
</style>
