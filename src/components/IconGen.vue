<template>
  <div ref="discordIconRootElement" id="icongen">

    <img :src="imageUri"/>

    <svg class="icon" :height="size" :width="size">
      <g>
        <mask id="mask-spikes">
          <rect width="100%" height="100%" fill="#000000"/>
          <path class="pathElement" fill="#FFFFFF"/>
          <path class="pathElement" fill="#FFFFFF"/>
          <path class="pathElement" fill="#FFFFFF"/>
          <path class="pathElement" fill="#FFFFFF"/>
          <ellipse fill="#000000" cx="64" cy="64" rx="63" ry="63"/>
        </mask>
        <mask id="mask-outer-ring">
          <ellipse fill="#FFFFFF" cx="64" cy="64" rx="63" ry="63"/>
          <ellipse fill="#000000" cx="64" cy="64" :rx="getRingSize1" :ry="getRingSize1"/>
        </mask>
        <mask id="mask-inner-ring">
          <ellipse fill="#FFFFFF" cx="64" cy="64" :rx="getRingSize1" :ry="getRingSize1"/>
          <ellipse fill="#000000" cx="64" cy="64" :rx="getRingSize2" :ry="getRingSize2"/>
        </mask>

      </g>
        <rect width="100%" height="100%" :fill="colors.outer.hex" mask="url(#mask-spikes)"/>
        <rect width="100%" height="100%" :fill="colors.outer.hex" mask="url(#mask-outer-ring)"/>
        <rect width="100%" height="100%" :fill="colors.inner.hex" mask="url(#mask-inner-ring)"/>
      <g>
      </g>
    </svg>
    <input v-model="imageURL" class="text" placeholder="image URL"/>
    <button v-on:click="retrieveImageFromURL()">get image from URL</button>
    <button v-on:click="exportToPng()" :disabled="!validUrl">export</button>

    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="width" :min="0" :max="640"/>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="height" :min="430" :max="640"/>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="ringSize" :min="0" :max="200"/>

    <chrome-picker v-model="colors.inner"/>
    <chrome-picker v-model="colors.outer"/>


  </div>
</template>

<script>
import vueSlider from 'vue-slider-component'
import saveSvgAsPng from "save-svg-as-png"
import jimp from "jimp"
import dataUriToBuffer from "data-uri-to-buffer"
import { Chrome } from "vue-color"

export default {
  name: 'IconGen',
  components: {
    vueSlider,
    'chrome-picker': Chrome
  },
  data () {
    return {
      width: 300,   // 0 - 64
      height: 615,  // 43 - 64
      size: 128,
      imageURL: 'https://i.imgur.com/em9oyLF.png',
      imageUri: 'https://i.imgur.com/nWxN3bH.png',
      ringSize: [50,100],
      validUrl: false, //todo: databind this to export button, if not valid disable button
      colors: {
        inner: {
					hex: '#2C2F33',
					hsl: {
						h: 144,
						s: 0,
						l: 1,
						a: 1
					},
					hsv: {
						h: 90,
						s: 0,
						v: 0,
						a: 1
					},
					rgba: {
						r: 255,
						g: 255,
						b: 255,
						a: 1
					},
					a: 1
				},
        outer: {
					hex: '#7289DA',
					hsl: {
						h: 227,
						s: 0.58,
						l: 0.65,
						a: 1
					},
					hsv: {
						h: 0,
						s: 0,
						v: 0,
						a: 1
					},
					rgba: {
						r: 114,
						g: 137,
						b: 218,
						a: 1
					},
					a: 1
				}
      }
    }
  },
  computed: {
    getRingSize1: {
      get: function() {
        return this.size/2 - this.ringSize[0]/6;
      }
    },
    getRingSize2: {
      get: function() {
        return this.size/2 - this.ringSize[1]/6;
      }
    }
  },
  methods: {
    exportToPng: function () {
      var discordIconRootElement = this.$refs.discordIconRootElement;
      var iconElement = discordIconRootElement.getElementsByClassName("icon")[0];

      var result_uri;
      var self = this

      saveSvgAsPng.svgAsPngUri(iconElement, {}, function(uri) {

        var buffer = dataUriToBuffer(uri);
        jimp.read(buffer).then(function (image) {

          var backgroundImageUrl = iconElement.style.backgroundImage.replace('url("', '').replace('")', '');

          jimp.read(backgroundImageUrl).then(function (jimpBackgroundImage) {
            jimp.read("https://i.imgur.com/nWxN3bH.png").then(function (blankImage) {
              jimpBackgroundImage.resize(128,128);
              jimpBackgroundImage.composite(image, 0, 0);
              blankImage.composite(jimpBackgroundImage, 0, 0)
              blankImage.getBase64(jimp.AUTO, function(err, new_uri) {
                  self.imageUri = new_uri;
              });
            });
          });
        });
      });
    },
    retrieveImageFromURL: function () {

      var discordIconRootElement = this.$refs.discordIconRootElement;
      var iconElement = discordIconRootElement.getElementsByClassName("icon")[0];
      iconElement.style.backgroundImage = "url(" + this.imageURL + ")";

      try {
        var http = new XMLHttpRequest();

        http.open('HEAD', this.imageURL, false);
        http.send();
      } catch(err) {
        this.validUrl = false;
      }

      if (http.status == 200) this.validUrl = true;
      else this.validUrl = false;
    },
    updateSpikes: function() {
      this.$nextTick(function() {
        var discordIconRootElement = this.$refs.discordIconRootElement;
        // todo: iterate over pathElement(s)
        var pathElements = discordIconRootElement.getElementsByClassName("pathElement");
        for (var j = 0; j < pathElements.length; j++) {
          var pathElement = pathElements[j];
          var d = "M 0,0 L 0,32 L 32,0 L 0,0 Z".split(" ");
          var new_d = []
          var counter = 0
          for (var i = 0; i < d.length; i++) {
            switch(d[i]) {
              case 'M':
              case 'C':
              case 'L':
              case 'Z':
                new_d.push(d[i]);

                break;
              default:
                var coordinates = d[i].split(',');
                var virtualWidth = 64 - this.width/10;
                var virtualHeight = 64 - this.height/10;

                if (counter == 0) {
                  if (j == 0)      new_d.push([virtualHeight,virtualHeight].join(","));
                  else if (j == 1) new_d.push([virtualHeight,this.size - virtualHeight].join(","));
                  else if (j == 2) new_d.push([this.size - virtualHeight,this.size - virtualHeight].join(","));
                  else if (j == 3) new_d.push([this.size - virtualHeight,virtualHeight].join(","));
                }
                if (counter == 1) {
                  if (j == 0)      new_d.push([64,virtualWidth].join(","));
                  else if (j == 1) new_d.push([64,this.size - virtualWidth].join(","));
                  else if (j == 2) new_d.push([64,this.size - virtualWidth].join(","));
                  else if (j == 3) new_d.push([64,virtualWidth].join(","));
                }
                if (counter == 2) {
                  if (j == 0)      new_d.push([virtualWidth,64].join(","));
                  else if (j == 1) new_d.push([virtualWidth,64].join(","));
                  else if (j == 2) new_d.push([this.size - virtualWidth,64].join(","));
                  else if (j == 3) new_d.push([this.size - virtualWidth,64].join(","));
                }
                if (counter == 3) {
                  if (j == 0)      new_d.push([virtualHeight,virtualHeight].join(","));
                  else if (j == 1) new_d.push([virtualHeight,this.size - virtualHeight].join(","));
                  else if (j == 2) new_d.push([this.size - virtualHeight,this.size - virtualHeight].join(","));
                  else if (j == 3) new_d.push([this.size - virtualHeight,virtualHeight].join(","));
                }
                counter++;
            }
          }
          pathElement.setAttribute("d", new_d.join(" "))
        }
      })
    }
  },
  watch: {
    height: function () {
      this.updateSpikes();
    },
    width: function() {
      this.updateSpikes();
    }
  },
  created: function () {
    this.updateSpikes();
    this.$nextTick(function () {
      this.retrieveImageFromURL();
    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#icongen {
  margin-top: 50px;
}

.icon {
  background-image: url(../assets/ROB_128.png);
  background-size: cover;
}
</style>
