<template>
  <div id="map" />

  <div id="widget-container">
    <button type="button" class="btn actions" @click="showModal('Actions')">
      Actions
    </button>

    <button type="button" class="btn layers" @click="showModal('Layers')">
      Layers
    </button>

    <button type="button" class="btn exp" @click="showModal('Export')">
      Export
    </button>
  </div>

  <div id="modal-container">
    <Modal v-show="isModalVisible" @close="closeModal" v-if="showActions">
      <template v-slot:header>
        Actions
      </template>

      <template v-slot:body>
        <button
          @click="selectedTab = 'icon'"
          :class="{ disabled: selectedTab === 'icon' }"
        >
          Icon
        </button>
        <button
          @click="selectedTab = 'text'"
          :class="{ disabled: selectedTab === 'text' }"
        >
          Text
        </button>
        <div v-if="selectedTab === 'icon'">
          <select
            v-model="selectedIcon"
            @click="markerMode = true"
            placeholder="Select Icon"
          >
            <option value="null" disabled hidden>Select Icon</option>
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
          <input type="text" v-model="selectedText" placeholder="Type here" />
          <button @click="markerMode = true">Submit</button>
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
            {{ layer.id }}--{{ layer.enabled }}
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
          <button class="btn" @click="exportMap">Download</button>
        </div>
      </template>
    </Modal>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import html2canvas from "html2canvas";
import Modal from "./components/Modal.vue";

export default {
  name: "App",
  components: {
    Modal,
  },
  data() {
    return {
      loading: false,
      access_token:
        "pk.eyJ1Ijoic21hcmFnaGkiLCJhIjoiY2p6aDdqcHVzMG1oODNtcGs1MTUzZTRlYSJ9.tL_9KWMY8Ft3Mb1eLFizHw",
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
      selectedText: "",
      selectedTab: "icon",
      isModalVisible: false,
    };
  },

  mounted() {
    this.createMap();
    this.loadData();
  },

  methods: {
    async createMap() {
      mapboxgl.accessToken = this.access_token;

      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/smaraghi/cks23g64y575p17p2stkczmta",
        center: [1.65, 40.93],
        zoom: 2,
        preserveDrawingBuffer: true,
      });

      const geolocate = new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true,
        },
        trackUserLocation: true,
      });

      const nav = new mapboxgl.NavigationControl();

      this.map.on("click", (e) => {
        if (this.markerMode === true) {
          const coordinates = e.lngLat;
          const el = document.createElement("div");

          if (this.selectedTab === "icon") {
            const width = 50;
            const height = 50;

            const yo = `/assets/icons/${this.selectedIcon}.svg`;
            el.style.backgroundImage = `url(${yo})`;

            el.className = "marker";
            el.style.width = `${width}px`;
            el.style.height = `${height}px`;
            el.style.backgroundSize = "100%";
          } else {
            el.innerText = `${this.selectedText}`;
          }

          new mapboxgl.Marker(el).setLngLat(coordinates).addTo(this.map);
          this.markerMode = false;
        }
      });

      this.map.addControl(geolocate, "top-right");
      this.map.addControl(nav, "top-right");
      this.map.on("load", () => {
        this.layerData = this.map.getStyle().layers.map((l) => {
          return { ...l, ...{ enabled: true } };
        });
      });
      return;
    },

    toggleLayer(layer) {
      const layerName = this.layerData.find((l) => l.id === layer.id);
      layerName.enabled = !layerName.enabled;
      layerName.isActive = !layerName.isActive;
      if (layerName.enabled === true) {
        this.map.setLayoutProperty(layerName.id, "visibility", "none");
      } else {
        this.map.setLayoutProperty(layerName.id, "visibility", "visible");
      }
    },

    exportMap() {
      console.log("yo ma export");
      const saveAs = (uri, filename) => {
        console.log("yo ma friend");
        var link = document.createElement("a");

        if (typeof link.download === "string") {
          link.href = uri;
          link.download = filename;

          //Firefox requires the link to be in the body
          document.body.appendChild(link);

          //simulate click
          link.click();

          //remove the link when done
          document.body.removeChild(link);
        } else {
          window.open(uri);
        }
      };

      const cnvs = document.querySelector("#map");
      html2canvas(cnvs, {
        width: this.exportWidth,
        height: this.exportHeight,
      }).then((canvas) => {
        saveAs(canvas.toDataURL(), "map.png");
      });
    },

    loadData() {
      // wouuld forloop through assets
      this.options = [
        { text: "Heart", value: "heart" },
        { text: "BBQ", value: "bbq" },
        { text: "Star", value: "star" },
      ];
    },

    showModal(type) {
      this.showActions = false;
      this.showLayers = false;
      this.showExport = false;

      if (type === "Actions") {
        this.showActions = true;
      } else if (type === "Layers") {
        this.showLayers = true;
      } else if (type === "Export") {
        this.showExport = true;
      }

      this.isModalVisible = true;
    },

    closeModal() {
      this.isModalVisible = false;
      this.showActions = false;
      this.showLayers = false;
      this.showExport = false;
    },

    activeTab(tab) {
      if (tab === "icon") {
        this.selectedTab === "icon";
      } else {
        this.selectedTab === "text";
      }
    },
  },
};
</script>

<style>
#map {
  height: 100vh;
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
  max-width: 500px;
  background: #fff;
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
/***
.modal__layer {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 200px;
  z-index: 2;
  background: gray;
  color: blue;
}

.modal__export {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 200px;
  z-index: 2;
  background: gray;
  color: blue;
}

.actions-modal {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 200px;
  z-index: 2;
  background: gray;
  color: blue;
}

**/
.active {
  color: red;
}

.disabled {
  background: gray;
}

#widget-container {
  position: absolute;
  display: flex;
  flex-direction: column;
  z-index: 10;
  top: 0;
  bottom: 0;
  max-height: 200px;
  justify-content: space-evenly;
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
  background-color: #415572;
  color: #fff;
  border: 1px solid #3c484d;
  border-radius: 2px;
}
</style>
