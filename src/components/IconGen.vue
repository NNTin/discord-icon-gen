<template>
  <div ref="discordIconRootElement" id="icongen">
    <svg class="icon" :height="size" :width="size">
      <g>
        <ellipse fill="#FF0000" stroke="#000000" cx="64" cy="64" rx="63" ry="63"/>
        <ellipse fill="#FFFF00" stroke="#000000" cx="64" cy="64" :rx="getRingSize1" :ry="getRingSize1"/>
        <ellipse fill="#FFFFFF" stroke="#000000" cx="64" cy="64" :rx="getRingSize2" :ry="getRingSize2"/>
      </g>
      <g>
        <path class="pathElement" fill="#0000FF" stroke="#000000" d="M 0,0 L 0,32 L 32,0 L 0,0 Z"/>
        <path class="pathElement" fill="#00FFFF" stroke="#000000" d="M 0,128 L 32,128 L 0,96 L 0,128 Z"/>
        <path class="pathElement" fill="#FF00FF" stroke="#000000" d="M 128,128 L 128,96 L 96,128 L 128,128 Z"/>
        <path class="pathElement" fill="#00FF00" stroke="#000000" d="M 128,0 L 96,0 L 128,32 L 128,0 Z"/>
      </g>
    </svg>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="width" :min="0" :max="640"/>
    <vueSlider ref="vueSlider" :reverse="false" direction="horizontal" :width="400" :height="10" v-model="height" :min="440" :max="640"/>
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
      width: 440,   // 0 - 64
      height: 550,  // 44 - 64
      size: 128,
      ringSize: [30,50]
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
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#icongen {
  margin-top: 50px;
}
</style>
