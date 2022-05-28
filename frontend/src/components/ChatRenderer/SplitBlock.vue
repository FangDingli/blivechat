<template>
  <div class="w100p h100p container">
    <div class="h100p block1" :style="{ backgroundColor: bgc }"></div>
    <div class="h100p block2" :style="{ backgroundColor: bgc }"></div>
  </div>
</template>

<script>
export default {
  props: {
    bgc: String,
  },
}
</script>

<style lang="less">
.container {
  display: flex;

  .block1 {
    width: 80%;
    margin-right: 10%;
  }
  .block2 {
    width: 10%;
  }
}
</style>
