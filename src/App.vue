<template>
  <div id="map" />

  <div id="widget-container">
    <button type="button" class="btn btn--widget" @click="showModal('Actions')">
      <svg class="icon"><use xlink:href="#icon-plus" /></svg>
    </button>

    <button type="button" class="btn btn--widget" @click="showModal('Layers')">
      <svg class="icon"><use xlink:href="#icon-layers" /></svg>
    </button>

    <button type="button" class="btn btn--widget" @click="showModal('Export')">
      <svg class="icon"><use xlink:href="#icon-share" /></svg>
    </button>
  </div>

  <div id="modal-container">
    <Modal v-show="isModalVisible" @close="closeModal" v-if="showActions">
      <template v-slot:header>
        Actions
      </template>

      <template v-slot:body>
        <div class="tabs">
          <button
            :class="{ disabled: selectedTab === 'icon' }"
            @click="selectedTab = 'icon'"
            class="tab-btn"
          >
            Icon
          </button>
          <button
            @click="selectedTab = 'text'"
            :class="{ disabled: selectedTab === 'text' }"
            class="tab-btn"
          >
            Text
          </button>
        </div>
        <div v-if="selectedTab === 'icon'">
          <select
            v-model="selectedIcon"
            @click="markerMode = true"
            placeholder="Select Icon"
          >
            <option value="null" disabled>Select Icon</option>
            <option
              v-for="option in options"
              v-bind:key="option"
              v-bind:value="option.value"
            >
              {{ option.text }}
            </option>
          </select>
        </div>
        <div v-if="selectedTab === 'text'">
          <div class="subheading">Submit your text, then click on the map</div>
          <input
            type="text"
            v-model="selectedText"
            placeholder="Add your text here..."
          />
          <button @click="markerMode = true" class="btn btn--submit">
            Submit
          </button>
        </div>
      </template>
    </Modal>

    <Modal v-show="isModalVisible" @close="closeModal" v-if="showLayers">
      <template v-slot:header>
        Layers
      </template>

      <template v-slot:body>
        <ul class="modal__layer">
          <li
            v-for="layer in layerData"
            :key="layer.id"
            :class="{ active: layer.isActive }"
            @click="toggleLayer(layer)"
          >
            {{ layer.id }}

            <template v-if="layer.isActive">
              <svg class="icon"><use xlink:href="#icon-eye" /></svg>
            </template>

            <template v-if="!layer.isActive">
              <svg class="icon"><use xlink:href="#icon-noeye" /></svg>
            </template>
          </li>
        </ul>
      </template>
    </Modal>

    <Modal v-show="isModalVisible" @close="closeModal" v-if="showExport">
      <template v-slot:header>
        Export
      </template>

      <template v-slot:body>
        <div class="modal__export">
          <p>Width</p>
          <input type="text" v-model="exportHeight" />
          <p>Height</p>
          <input type="text" v-model="exportWidth" />
          <button class="btn btn--download" @click="exportMap">Download</button>
        </div>
      </template>
    </Modal>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
import html2canvas from 'html2canvas'
import Modal from './components/Modal.vue'

export default {
  name: 'App',
  components: {
    Modal
  },
  data() {
    return {
      loading: false,
      access_token:
        'pk.eyJ1Ijoic21hcmFnaGkiLCJhIjoiY2p6aDdqcHVzMG1oODNtcGs1MTUzZTRlYSJ9.tL_9KWMY8Ft3Mb1eLFizHw',
      center: [0, 0],
      map: {},
      showActions: false,
      showLayers: false,
      showExport: false,
      exportHeight: 760,
      exportWidth: 1080,
      layerData: [],
      markerMode: false,
      options: [],
      selectedIcon: null,
      selectedText: '',
      selectedTab: 'icon',
      isModalVisible: false,
      publicPath: process.env.BASE_URL
    }
  },

  mounted() {
    this.createMap()
    this.loadData()
  },

  methods: {
    async createMap() {
      mapboxgl.accessToken = this.access_token

      this.map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/smaraghi/cks23g64y575p17p2stkczmta',
        center: [1.65, 40.93],
        zoom: 2,
        preserveDrawingBuffer: true
      })

      const geolocate = new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true
        },
        trackUserLocation: true
      })

      const nav = new mapboxgl.NavigationControl()

      this.map.on('click', (e) => {
        if (this.markerMode === true) {
          const coordinates = e.lngLat
          const el = document.createElement('div')
          el.className = 'marker'

          if (this.selectedTab === 'icon') {
            const icon = `${this.publicPath}assets/icons/${this.selectedIcon}.svg`
            el.style.backgroundImage = `url(${icon})`
          } else {
            el.innerText = `${this.selectedText}`
          }

          new mapboxgl.Marker(el).setLngLat(coordinates).addTo(this.map)
          this.markerMode = false
        }
      })

      this.map.addControl(geolocate, 'top-right')
      this.map.addControl(nav, 'top-right')
      this.map.on('load', () => {
        this.layerData = this.map.getStyle().layers.map((l) => {
          return { ...l, ...{ enabled: true } }
        })
      })
    },

    toggleLayer(layer) {
      const layerName = this.layerData.find((l) => l.id === layer.id)
      layerName.enabled = !layerName.enabled
      layerName.isActive = !layerName.isActive
      if (layerName.isActive === true) {
        this.map.setLayoutProperty(layerName.id, 'visibility', 'none')
      } else {
        this.map.setLayoutProperty(layerName.id, 'visibility', 'visible')
      }
    },

    exportMap() {
      const saveAs = (uri, filename) => {
        let link = document.createElement('a')

        if (typeof link.download === 'string') {
          link.href = uri
          link.download = filename

          document.body.appendChild(link)

          link.click()

          document.body.removeChild(link)
        } else {
          window.open(uri)
        }
      }

      const cnvs = document.querySelector('#map')
      html2canvas(cnvs, {
        width: this.exportWidth,
        height: this.exportHeight
      }).then((canvas) => {
        saveAs(canvas.toDataURL(), 'map.png')
      })
    },

    loadData() {
      // iterate through assets using webpack
      this.options = [
        { text: 'Park', value: 'park' },
        { text: 'School', value: 'school' },
        { text: 'Star', value: 'star' },
        { text: 'Stadium', value: 'stadium' },
        { text: 'Garden', value: 'garden' }
      ]
    },

    showModal(type) {
      this.showActions = false
      this.showLayers = false
      this.showExport = false

      if (type === 'Actions') {
        this.showActions = true
      } else if (type === 'Layers') {
        this.showLayers = true
      } else if (type === 'Export') {
        this.showExport = true
      }

      this.isModalVisible = true
    },

    closeModal() {
      this.isModalVisible = false
      this.showActions = false
      this.showLayers = false
      this.showExport = false
    },

    activeTab(tab) {
      if (tab === 'icon') {
        this.selectedTab === 'icon'
      } else {
        this.selectedTab === 'text'
      }
    }
  }
}
</script>

<style>
#map {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
  z-index: -5;
}

.modal {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  z-index: 50;
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 350px;
  background: rgba(255, 255, 255);
  box-shadow: 0 6px 8px 0 rgba(44, 8, 8, 0.5);
}

.modal-fade-enter,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.7s ease;
}

select {
  max-width: 8.5625rem;
  min-width: 250px;
  padding: 0.5rem 2rem 0.5rem 0.75rem;
  margin-top: 16px;
  border-radius: 2px;
  background: rgba(255, 255, 255);
  border: 1px solid #ddd;
}

#widget-container {
  position: absolute;
  display: flex;
  flex-direction: column;
  margin: 2rem 1.5rem 0;
  z-index: 10;
  top: 0;
  bottom: 0;
  max-height: 50px;
}

input {
  padding: 0.5rem 4rem 0.5rem 1rem;
  max-width: 150px;
  overflow: hidden;
  border: 1px solid #ddd;
  border-radius: 1px;
  box-shadow: 0 0 0 1px rgb(0 0 0 / 10%);
}

.btn {
  font-weight: 500;
  line-height: 1.37;
  letter-spacing: 0.2px;
  text-align: center;
  font-size: 14px;
  display: inline-block;
  padding: 8px 16px;
  text-align: center;
  border-radius: 4px;
  background: #fff;
  border: 1px solid #ddd;
  box-shadow: 0 0 0 1px rgb(0 0 0 / 10%);
  cursor: pointer;
}

.btn:hover {
  background: rgba(255, 255, 255, 0.9);
}

.btn--widget {
  padding: 2px 6px;
  margin-bottom: 8px;
}

.btn--widget svg {
  width: 20px;
  height: 20px;
}

.mapboxgl-ctrl-top-right {
  right: unset;
  left: 2rem;
  top: 10rem;
}

.mapboxgl-ctrl-top-right button {
  width: 33px;
}

.btn--download {
  margin-top: 24px;
  max-width: fit-content;
}

.btn--submit {
  margin-top: 12px;
  margin-left: 8px;
  padding: 6px 24px;
  border-radius: 2px;
}

.tab-btn {
  background: #202c36;
  color: #fff;
  border: none;
}

.disabled {
  background: #fff;
  color: rgba(32, 44, 54, 0.87);
  outline: none;
  border: none;
}

.tabs {
  position: relative;
  top: 0;
  z-index: 5;
  display: flex;
  margin: 0 -2rem 3rem;
}

.tabs button {
  position: relative;
  flex: 1 0 50%;
  padding: 1rem 2rem;
  text-transform: uppercase;
  cursor: pointer;
}

.icon {
  width: 25px;
  height: 25px;
}

.modal__layer {
  list-style-type: none;
}

.modal__layer li {
  display: flex;
  align-items: center;
  text-transform: capitalize;
}

.modal__layer li svg {
  margin-left: 8px;
}

.modal__export {
  display: flex;
  flex-direction: column;
}

.marker {
  height: 50px;
  width: 50px;
  font-size: 14px;
  background-size: 100%;
  stroke: rgba(255, 255, 255);
  fill: rgba(255, 255, 255);
  filter: invert(100%) sepia(0%) saturate(7485%) hue-rotate(1deg)
    brightness(100%) contrast(103%);
}

.subheading {
  margin-bottom: 12px;
  font-size: 16px;
  color: #202c36;
}
</style>
