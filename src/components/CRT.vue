<template>
  <canvas :width="screenW" :height="screenH" />
</template>

<script>
import {runCrtAnimation} from "../methods/CrtAnimation";

export default {
  name: "CRT",
  data: () => {
    return {
      screenH: window.innerHeight > 1000 ? 1000 : window.innerHeight,
      screenW: 1000,
    };
  },
  mounted() {
    runCrtAnimation();
  },
};
</script>

<style scoped>
div {
  border-radius: 40px;
  box-shadow: 0 0 50px 50px #000;
  background: #000;
}
</style>
