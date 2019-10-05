<template>
  <div id="progress">
    <div :style="{ width: 100 - (100 / max * value) + '%' }"></div>
  </div>
</template>

<script>
let timer = null
export default {
  props: ['value', 'max'],
}
</script>

<style scoped>
#progress {
  background: #cee6ce;
  border-radius: 999px;
  width: 100%;
  height: 36px;
  margin: 20px 0;
  overflow: hidden;
  border: 2px solid #00844a;
}

#progress div {
  background: #00844a;
  height: inherit;
  transition: width ease-out 100ms;
}
</style>
