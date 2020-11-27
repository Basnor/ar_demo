<template>
  <div class="model">
    <model-viewer 
      :src="`FlightHelmet/FlightHelmet.gltf`" 
      shadow-intensity="1" 
      ar 
      ar-scale="auto" 
      camera-controls 
      alt="A 3D model carousel"> 

        <button slot="ar-button" id="ar-button">
          View in your space
        </button>

        <div id="ar-prompt">
          <img :src="require(`./assets/hand.png`)">
        </div>

        <div class="slider">
          <div class="slides">
            <div v-if="sliderItems.length > 1">
              <button class="slide" @click="back()">Назад</button>
            </div>
            <div v-for="item in sliderItems.slice(-1)[0]" :key="item.id">
              <button 
                class="slide" 
                v-bind:style="{'background-color': 'rgb(' + item.color + ')', 
                  'background-image': 'url(' + item.url + ')'}" 
                @click="clicked(item)">{{item.name}}
              </button>
            </div>
          </div>
        </div>
    </model-viewer>
  </div>
</template>

<script>
import '@google/model-viewer'
import options from './sliderOptions.json'

export default {
  name: 'Model',
  data() {
    return {
      sliderItems: [options],
      material: ''
    }
  },
  methods: {
    back() {
      this.sliderItems.splice(-1, 1);
    },
    clicked(item) {
      if (item.configs) {
        this.sliderItems.push(item.configs);
        return;
      } 
      
      const modelViewer = document.querySelector("model-viewer");
      this.material = modelViewer.model.materials[item.mat_index];
      /*console.log(this.material);*/

      if (item.url && item.channel === 'baseColorTexture') {
        this.applyPBRTexture(item.channel, item.url);
      }

      if (item.url && item.channel === 'normalTexture') {
        this.applyTexture(item.channel, item.url);
      }

      if (item.color) {
        this.applyRGBAColor(item.color);
      }
      
    },
    applyPBRTexture(channel, url) {
      this.material.pbrMetallicRoughness[channel].texture.source.setURI(url);
    },
    applyRGBAColor(strColor) {
      strColor += ',255';
      const color = strColor.split(',')
            .map(numberString => parseFloat(numberString) / 255);
      this.material.pbrMetallicRoughness.setBaseColorFactor(color);
    },
    applyTexture(channel, url) {
      this.material[channel].texture.source.setURI(url);
    }
  }
}
</script>

<style>
/* This keeps child nodes hidden while the element loads */
:not(:defined) > * {
  display: none;
}

model-viewer {
  background-color: #eee;
  --poster-color: #eee;
}

#ar-button {
  background-image: url(./assets/ic_view_in_ar_new_googblue_48dp.png);
  background-repeat: no-repeat;
  background-size: 20px 20px;
  background-position: 12px 50%;
  background-color: #fff;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  bottom: 132px;
  padding: 0px 16px 0px 40px;
  font-family: Roboto Regular, Helvetica Neue, sans-serif;
  font-size: 14px;
  color:#4285f4;
  height: 36px;
  line-height: 36px;
  border-radius: 18px;
  border: 1px solid #DADCE0;
}

#ar-button:active {
  background-color: #E8EAED;
}

#ar-button:focus {
  outline: none;
}

#ar-button:focus-visible {
  outline: 1px solid #4285f4;
}

@keyframes circle {
  from { transform: translateX(-50%) rotate(0deg) translateX(50px) rotate(0deg); }
  to   { transform: translateX(-50%) rotate(360deg) translateX(50px) rotate(-360deg); }
}

@keyframes elongate {
  from { transform: translateX(100px); }
  to   { transform: translateX(-100px); }
}

model-viewer > #ar-prompt {
  position: absolute;
  left: 50%;
  bottom: 175px;
  animation: elongate 2s infinite ease-in-out alternate;
  display: none;
}

model-viewer[ar-status="session-started"] > #ar-prompt {
  display: block;
}

model-viewer > #ar-prompt > img {
  animation: circle 4s linear infinite;
}

.slider {
  width: 100%;
  text-align: center;
  overflow: hidden;
  position: absolute;
  bottom: 16px;
}

.slides {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

.slide {
  scroll-snap-align: start;
  flex-shrink: 0;
  width: 100px;
  height: 100px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-color: #fff;
  margin-right: 10px;
  border-radius: 10px;
  border: none;
  display: flex;
}

.slide.selected {
  border: 2px solid #4285f4;
}

.slide:focus {
  outline: none;
  border: 2px solid #4285f4;
}

.slide:focus-visible {
  outline: 1px solid #4285f4;
}

.model {
  position: sticky;
  top: 0;
  height: 100vh;
  flex: 1;
  display: flex;
  justify-content: center;
  box-sizing: border-box;
}

.model model-viewer {
  width: 100%;
  height: 100%;
  background-color: #eee;
}

@media only screen and (max-width: 800px) {  
  .model {
    position: relative;
    height: 76vh;
    flex-direction: column-reverse;
    background-color: #455A64;
  }
}
</style>
