<template>
  <div ref="discordIconRootElement" id="icongen">
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
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="width" :min="0" :max="640"/>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="height" :min="430" :max="640"/>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="ringSize" :min="0" :max="100"/>
  </div>
</template>

<script>
import vueSlider from 'vue-slider-component'

export default {
  name: 'IconGen',
  components: {
    vueSlider
  },
  data () {
    return {
      width: 200,   // 0 - 64
      height: 550,  // 43 - 64
      size: 128,
      ringSize: [30,50],
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
}
</style>
