<template>
  <div id="app">
    <div class="content">
      <div class="crt">
        <CRT />
      </div>
      <div class="product">
        <HelloWorld />
        <ProductInfo />
      </div>
    </div>
  </div>
</template>

<script>
import CRT from "./components/CRT.vue";
import HelloWorld from "./components/HelloWorld.vue";
import ProductInfo from "./components/ProductInfo.vue";

export default {
  name: "App",
  components: {
    HelloWorld,
    CRT,
    ProductInfo,
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
  font-family: "Playfair Display", serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #88aa93;
  font-size: 20px;
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
