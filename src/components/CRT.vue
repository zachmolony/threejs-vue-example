<template>
  <canvas :width="screenW" :height="screenH" />
</template>

<script>
export default {
  name: "CRT",
  data: () => {
    return {
      screenH: window.innerHeight > 1000 ? 1000 : window.innerHeight,
      screenW: window.innerWidth > 1000 ? 1000 : window.innerWidth,
    };
  },
  mounted() {
    this.runWebGL();
  },
  methods: {
    runWebGL() {
      var one = {color: ""};
      one.color = require("onecolor");

      const onecolor = one.color;

      const gl = require("gl-matrix");

      function hex2vector(cssHex) {
        const pc = onecolor(cssHex);

        return gl.vec3.fromValues(pc.red(), pc.green(), pc.blue());
      }

      const charW = 6;
      const charH = 10;
      const bufferCW = 80;
      const bufferCH = 24;
      const bufferW = bufferCW * charW;
      const bufferH = bufferCH * charH;
      const textureW = 512;
      const textureH = 256;

      const consolePad = 8; // in texels
      const consoleW = bufferW + consolePad * 2;
      const consoleH = bufferH + consolePad * 2;

      const bufferCanvas = document.createElement("canvas");
      bufferCanvas.width = bufferW;
      bufferCanvas.height = bufferH;
      // document.body.appendChild(bufferCanvas);

      const bufferContext = bufferCanvas.getContext("2d");

      bufferContext.fillStyle = "#000";
      bufferContext.fillRect(0, 0, bufferW, bufferH);

      function charRange(start, end) {
        return Array.apply(null, new Array(end - start)).map((_, index) => {
          return String.fromCharCode(start + index);
        });
      }

      const characterSet = []
        .concat(charRange(0x30, 0x3a)) // ASCII digits
        //  .concat(charRange(0x40, 0x5b)) // ASCII uppercase and @
        .concat(charRange(0x30a0, 0x30ff)); // kanji

      // pseudo-random
      // credit: https://gist.github.com/blixt/f17b47c62508be59987b
      const SEED_OFFSET = new Date().getTime();

      function randomize(seed) {
        const intSeed = seed % 2147483647;
        const safeSeed = intSeed > 0 ? intSeed : intSeed + 2147483646;
        return (safeSeed * 16807) % 2147483647;
      }

      function getRandomizedFraction(seed) {
        return (seed - 1) / 2147483646;
      }

      // main character trail state
      function createTrail() {
        const cx = Math.floor(Math.random() * bufferCW);
        const cy = 0;
        const cvy = 5 + Math.random() * 15;

        return [cx, cy, cvy];
      }

      const trails = Array.apply(null, new Array(10)).map((_, index) => {
        return createTrail();
      });

      function updateWorld(delta) {
        trails.forEach((trail, index) => {
          trail[1] += trail[2] * delta;

          if (trail[1] > bufferCH) {
            trails[index] = createTrail();
          }
        });
      }

      // "warm up" the state by simulating the world for a bit
      Array.apply(null, new Array(100)).forEach(() => {
        updateWorld(0.1);
      });

      let fadeCountdown = 0;
      let bannerCountdown = 8.0;
      let bannerChoice = 0;

      function renderWorld(delta) {
        // fade screen every few frames
        // (not every frame, for long trails without rounding artifacts)
        fadeCountdown -= delta;

        // let insert_coin = new Image();
        // insert_coin.src = require("@/assets/insert_coin.png");

        let img = new Image();
        img.src = require("@/assets/logo.png");

        // let arrowL = new Image();
        // arrowL.src = require("@/assets/arrowL.png");

        // let arrowR = new Image();
        // arrowR.src = require("@/assets/arrowR.png");

        if (fadeCountdown < 0) {
          bufferContext.fillStyle = "rgba(0, 0, 0, 0.5)";
          bufferContext.fillRect(0, 0, bufferW, bufferH);

          bufferContext.drawImage(img, 95, -30, 300, 120);

          // bufferContext.drawImage(insert_coin, 150, 145, 130, 17);

          // bufferContext.drawImage(arrowL, 150, 145, 130, 17);

          // bufferContext.drawImage(arrowR, 150, 145, 130, 17);

          fadeCountdown += 0.2;
        }

        // redraw
        bufferContext.textAlign = "center";
        bufferContext.font = '8px "Inconsolata"';

        //*
        trails.forEach((trail, index) => {
          const k = index / trails.length;
          const charY = Math.floor(trail[1]);

          // randomize based on character position
          const charSeed = index + (trail[0] + charY * bufferCW) * 50;
          const outSeed = randomize(charSeed * 1500 + SEED_OFFSET);

          const char =
            characterSet[
              Math.floor(getRandomizedFraction(outSeed) * characterSet.length)
            ];

          bufferContext.fillStyle = `hsl(${120 + k * 2}, 100%, 30%)`;
          bufferContext.fillText(
            char,
            (trail[0] + 0.5) * charW, // center inside character box
            charY * charH + charH,
            charW // restrict width, but allow a tiny bit of spillover
          );
          //
        });

        /*
        I;
        // fade screen every few frames
        // (not every frame, for long trails without rounding artifacts)
        bannerCountdown -= delta;

        if (bannerCountdown < 1.5) {
          bufferContext.fillStyle = `hsla(${180 +
            Math.random() * 220}, 100%, 30%, 1)`;
          bufferContext.fillRect(0, 0, bufferW, bufferH);

          if (bannerCountdown > 0.8) {
            bannerChoice = Math.floor(Math.random() * bannerImgSet.length);
          }

          const bannerImg = bannerImgSet[bannerChoice];
          bufferContext.drawImage(bannerImg, 0, 10, bufferW, bufferH + 40);

          // bufferContext.fillStyle = `hsla(${100 + Math.random() * 220}, 100%, 10%, 1)`;
          // bufferContext.font = '250px sans-serif';

          // bufferContext.save();
          // bufferContext.scale(2, 1);
          // bufferContext.fillText(
          //   bannerSet[Math.floor(Math.random() * bannerSet.length)],
          //   240 / 2, 200
          // );
          // bufferContext.restore();
        }

        if (bannerCountdown < 0) {
          bannerCountdown += 10 + Math.random() * 10;
        }
      */
      }

      // init WebGL
      const regl = require("regl")({
        canvas: document.body.querySelector("canvas"),
        attributes: {
          antialias: true,
          alpha: false,
          preserveDrawingBuffer: true,
        },
      });

      const spriteTexture = regl.texture({
        width: 512,
        height: 256,
        mag: "linear",
      });

      const termFgColor = hex2vector("#145959");
      const termBgColor = hex2vector("#001a1a");

      const quadCommand = regl({
        vert: `
                    precision mediump float;

                    attribute vec3 position;

                    varying vec2 uvPosition;

                    void main() {
                        uvPosition = position.xy * vec2(0.5, -0.5) + vec2(0.5);

                        gl_Position = vec4(
                            vec2(-1.0, 1.0) + (position.xy - vec2(-1.0, 1.0)) * 1.0,
                            0.0,
                            1.0
                        );
                    }
                `,

        frag: `
                    precision mediump float;

                    varying vec2 uvPosition;
                    uniform sampler2D sprite;
                    uniform float time;
                    uniform vec3 bgColor;
                    uniform vec3 fgColor;

                    #define textureW ${textureW + ".0"}
                    #define textureH ${textureH + ".0"}
                    #define consoleW ${consoleW + ".0"}
                    #define consoleH ${consoleH + ".0"}
                    #define consolePadUVW ${consolePad / consoleW}
                    #define consolePadUVH ${consolePad / consoleH}
                    #define charUVW ${charW / consoleW}
                    #define charUVH ${charH / consoleH}

                    void main() {
                        // @todo use uniform
                        vec2 consoleWH = vec2(consoleW, consoleH);

                        // @todo use uniforms
                        float glitchLine = mod(0.8 + time * 0.07, 1.0);
                        float glitchFlutter = mod(time * 40.0, 1.0); // timed to be slightly out of sync from main frame rate
                        float glitchAmount = 0.06 + glitchFlutter * 0.001;
                        float glitchDistance = 0.04 + glitchFlutter * 0.15;

                        vec2 center = uvPosition - vec2(0.5);
                        float factor = dot(center, center) * 0.2;
                        vec2 distortedUVPosition = uvPosition + center * (1.0 - factor) * factor;

                        vec2 fromEdge = vec2(0.5, 0.5) - abs(distortedUVPosition - vec2(0.5, 0.5));

                        if (fromEdge.x > 0.0 && fromEdge.y > 0.0) {
                            vec2 fromEdgePixel = min(0.2 * consoleWH * fromEdge, vec2(1.0, 1.0));

                            // simulate 2x virtual pixel size, for crisp display on low-res
                            vec2 inTexel = mod(distortedUVPosition * consoleWH * 0.5, vec2(1.0));

                            float distToGlitch = glitchLine - (distortedUVPosition.y - inTexel.y / consoleH);
                            float glitchOffsetLinear = step(0.0, distToGlitch) * max(0.0, glitchDistance - distToGlitch) / glitchDistance;
                            float glitchOffset = glitchOffsetLinear * glitchOffsetLinear;

                            vec2 inTexelOffset = inTexel - 0.5;
                            float scanlineAmount = inTexelOffset.y * inTexelOffset.y / 0.25;
                            float intensity = 8.0 - scanlineAmount * 5.0 + glitchOffset * 2.0; // ray intensity is over-amped by default
                            vec2 uvAdjustment = inTexelOffset * vec2(0.0, .5 / consoleH); // remove vertical texel interpolation

                            distortedUVPosition.x -= glitchOffset * glitchAmount + 0.011 * (glitchFlutter * glitchFlutter * glitchFlutter);

                            vec4 sourcePixel = texture2D(
                                sprite,
                                (distortedUVPosition - uvAdjustment) * consoleWH / vec2(textureW, textureH)
                            );

                            vec3 pixelRGB = sourcePixel.rgb * sourcePixel.a;

                            // multiply by source alpha as well
                            float screenFade = 1.0 - dot(center, center) * 1.8;
                            float edgeFade = fromEdgePixel.x * fromEdgePixel.y;
                            gl_FragColor = vec4(edgeFade * screenFade * mix(
                                bgColor,
                                fgColor,
                                intensity * pixelRGB + glitchOffset * 1.5
                            ) * (1.0 - 0.2 * scanlineAmount), 0.2);
                        } else {
                            gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
                        }
                    }
                `,

        attributes: {
          position: regl.buffer([
            [-1, -1, 0],
            [1, -1, 0],
            [-1, 1, 0],
            [1, 1, 0],
          ]),
        },

        uniforms: {
          time: regl.context("time"),
          camera: regl.prop("camera"),
          sprite: spriteTexture,
          bgColor: regl.prop("bgColor"),
          fgColor: regl.prop("fgColor"),
        },

        primitive: "triangle strip",
        count: 4,

        depth: {
          enable: false,
        },

        blend: {
          enable: true,
          func: {
            src: "src alpha",
            dst: "one minus src alpha",
          },
        },
      });

      regl.clear({
        depth: 1,
        color: [0, 0, 0, 1],
      });

      // main loop
      let currentTime = performance.now();

      function rafBody() {
        // measure time
        const newTime = performance.now();
        const delta = Math.min(0.05, (newTime - currentTime) / 1000); // apply limiter to avoid frame skips
        currentTime = newTime;

        updateWorld(delta);
        renderWorld(delta);

        regl.poll();
        spriteTexture.subimage(bufferContext, consolePad, consolePad);
        quadCommand({
          bgColor: termBgColor,
          fgColor: termFgColor,
        });

        requestAnimationFrame(rafBody);
      }

      // kickstart the loop
      rafBody();
    },
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
