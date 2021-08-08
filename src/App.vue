<template>
  <button id="yo" class="yo" @click="toggleF = !toggleF">yo</button>
  <button class="layers" @click="showLayers = !showLayers">Layers</button>
  <div>{{ toggleF }}</div>
  <div id="map" />

  <div class="layer-modal" v-if="showLayers">
    <ul>
      <li
        v-for="layer in layerData"
        :key="layer.id"
        :class="{ active: layer.isActive }"
        @click="toggleLayer(layer)"
      >
        {{ layer.id }}--{{ layer.enabled }}
      </li>
    </ul>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  data() {
    return {
      loading: false,
      access_token:
        "pk.eyJ1Ijoic21hcmFnaGkiLCJhIjoiY2p6aDdqcHVzMG1oODNtcGs1MTUzZTRlYSJ9.tL_9KWMY8Ft3Mb1eLFizHw",
      center: [0, 0],
      map: {},
      toggleF: true,
      showLayers: false,
      layerData: [],
    };
  },

  mounted() {
    this.createMap();
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
        if (this.toggleF === true) {
          const coordinates = e.lngLat;
          const el = document.createElement("div");

          const width = 50;
          const height = 50;

          //           .marker {
          //   background-image: url("~@/assets/heart.svg");
          // }
          // el.style.backgroundImage = `url(https://placekitten.com/g/${width}/${height}/)`;
          // el.style.backgroundImage = `url("/assets/heart.svg")`;
          el.style.backgroundImage = `url("/assets/logo.png")`;
          // this.svg = require(`resources/assets/images/svg/${this.name}.svg`);
          // el.style.backgroundImage = this.svg;
          el.className = "marker";
          el.style.width = `${width}px`;
          el.style.height = `${height}px`;
          el.style.backgroundSize = "100%";
          new mapboxgl.Marker(el).setLngLat(coordinates).addTo(this.map);
          console.log(el);
        }
      });

      this.map.addControl(geolocate, "top-right");
      this.map.addControl(nav, "top-right");
      this.map.on("load", () => {
        this.layerData = this.map.getStyle().layers.map((l) => {
          return { ...l, ...{ enabled: true } };
        });
        console.log(this.layerData);
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
  },
};
</script>

<style>
#map {
  height: 100vh;
}

.layer-modal {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 200px;
  z-index: 2;
  background: gray;
  color: blue;
}

.active {
  color: red;
}
</style>
