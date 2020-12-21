<template>
  <div class="about">
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from '@/components/map-about/index.vue';
export default {
  name: 'About',
  components: {
    HelloWorld
  }
};
</script>

<style lang="less" scoped>
.about {
  margin: 0 auto;
  background: url('../assets/bac.png') 0 0 no-repeat;
  width: 1200px;
  height: 600px;
}
</style>
