<template>
  <div id="scene-container" ref="sceneContainer"></div>
</template>

<script>
import * as THREE from "three";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";
import {GLTFLoader} from "three/examples/jsm/loaders/GLTFLoader";
import Stats from "stats.js";

export default {
  name: "HelloWorld",
  data() {
    return {
      container: null,
      scene: null,
      camera: null,
      controls: null,
      renderer: null,
      stats: null,
    };
  },
  methods: {
    init() {
      // set container
      this.container = this.$refs.sceneContainer;

      // add stats
      this.stats = new Stats();
      this.container.appendChild(this.stats.dom);

      // add camera
      const fov = 10; // Field of view
      const aspect = this.container.clientWidth / this.container.clientHeight;
      const near = 0.1; // the near clipping plane
      const far = 70; // the far clipping plane
      const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
      camera.position.set(0, 5, 10);
      this.camera = camera;

      // create scene
      this.scene = new THREE.Scene();
      // this.scene.background = new THREE.Color('skyblue')

      // add lights
      const ambientLight = new THREE.HemisphereLight(
        0xffffff, // bright sky color
        0x222222, // dim ground color
        1 // intensity
      );
      const mainLight = new THREE.DirectionalLight(0xffffff, 4.0);
      mainLight.position.set(10, 10, 10);
      this.scene.add(ambientLight, mainLight);

      // add controls
      this.controls = new OrbitControls(this.camera, this.container);
      this.controls.autoRotate = true;

      // create renderer
      this.renderer = new THREE.WebGLRenderer({antialias: true, alpha: true});
      this.renderer.setSize(
        this.container.clientWidth,
        this.container.clientHeight
      );
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.gammaFactor = 2.2;
      this.renderer.outputEncoding = THREE.sRGBEncoding;
      this.renderer.physicallyCorrectLights = true;
      this.container.appendChild(this.renderer.domElement);

      // set aspect ratio to match the new browser window aspect ratio
      this.camera.aspect =
        this.container.clientWidth / this.container.clientHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(
        this.container.clientWidth,
        this.container.clientHeight
      );

      const loader = new GLTFLoader();

      loader.load(
        "/three-assets/HoodieBase.gltf",
        (gltf) => {
          this.mesh = gltf;
          this.mesh.scene.position.y = -1;
          this.scene.add(gltf.scene);

          gltf.animations; // Array<THREE.AnimationClip>
          gltf.scene; // THREE.Group
          gltf.scenes; // Array<THREE.Group>
          gltf.cameras; // Array<THREE.Camera>
          gltf.asset; // Object
        },
        undefined,
        undefined
      );

      this.renderer.setAnimationLoop(() => {
        this.render();
        this.controls.update();
      });
    },
    render() {
      this.renderer.render(this.scene, this.camera);
      this.stats.update();
      // this.animate();
    },
    animate: function() {
      requestAnimationFrame(this.animate);
      this.mesh.rotation.y += 0.02;
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#scene-container {
  height: 100%;
}
</style>
