<template>
  <div id="app">
    <CRT />
    <div class="container">
      <HelloWorld />
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
html,
body,
#app {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: blue;
}
.container {
  width: 900px;
  height: 900px;
}
</style>
