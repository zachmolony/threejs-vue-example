<template>
  <div id="app">
    <div class="content">
      <div class="crt">
        <CRT />
      </div>
      <div class="product">
        <HelloWorld />
      </div>
    </div>
  </div>
</template>

<script>
import CRT from "./components/CRT.vue";
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "App",
  components: {
    HelloWorld,
    CRT,
  },
};
</script>

<style>
body {
  background-color: black;
  height: 100%;
  width: 100%;
}
html,
body,
#app {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  position: relative;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.content {
  width: 1000px;
  height: 1000px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.product {
  margin-top: 100px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.crt {
  position: absolute;
  z-index: -1;
}
</style>
