<template>
  <div class="map-container">
    <l-map
      ref="map"
      class="map"
      v-model:zoom="zoom"
      :center="mapLatLng"
      @ready="onMapReady"
      @locationfound="onLocationFound"
    >
      <l-tile-layer
        url="https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png'"
        layer-type="cyclosm"
        name="OpenStreetMap"
        :min-zoom="getMinZoom"
      ></l-tile-layer>
      <l-control position="topleft">
        <q-btn
          round
          color="secondary"
          icon="my_location"
          @click="onLocationClick"
        ></q-btn>
      </l-control>
      <l-control-scale
        position="topright"
        :imperial="true"
        :metric="true"
      ></l-control-scale>
      <l-marker v-if="markerLatLng" :lat-lng="markerLatLng" />
      <l-circle
        v-if="markerLatLng"
        :lat-lng="markerLatLng"
        :radius="markerSize"
        color="teal"
      />
    </l-map>
  </div>
</template>

<script>
import "leaflet";
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LCircle,
  LMarker,
  LControl,
  LControlScale,
} from "@vue-leaflet/vue-leaflet";

const MAP_LAT_LNG_DEFAULT = [35.0238, -80.9879];

class Deferred {
  constructor() {
    this.promise = new Promise((resolve, reject) => {
      this.reject = reject;
      this.resolve = resolve;
    });
  }
}

export default {
  name: "IndexPage",
  components: {
    LMap,
    LTileLayer,
    LCircle,
    LMarker,
    LControl,
    LControlScale,
  },
  emits: ["location"],
  beforeMount() {
    this.zoom = this.getMinZoom;
  },
  methods: {
    onMapReady(map) {
      this.map = map;
      this.map.on("locationfound", this.onLocationFound);
      this.getLocation().then(this.recenterMap);
    },
    onLocationClick() {
      this.getLocation().then(this.recenterMap);
    },
    getLocation() {
      this.map.locate({ watch: true });
      this.locationResolver = new Deferred();
      return this.locationResolver.promise;
    },
    onLocationFound(l) {
      this.markerSize = l.accuracy;
      this.markerLatLng = l.latlng;

      if (this.locationResolver && this.locationResolver.resolve) {
        this.locationResolver.resolve();
        this.locationResolver = null;
      }

      this.$emit("location", this.markerLatLng);
    },
    recenterMap() {
      if (this.markerLatLng)
        this.map.setView(this.markerLatLng, 20, { animation: true });
    },
  },
  computed: {
    getMinZoom() {
      return this.$q.platform.is.mobile ? 13 : 15;
    },
  },
  data() {
    return {
      map: null,
      zoom: 15,
      mapLatLng: MAP_LAT_LNG_DEFAULT,
      markerLatLng: false,
      markerSize: 0,
      locationResolver: null,
    };
  },
};
</script>
<style scoped>
body {
  position: fixed;
  height: 100%;
}
.map-container {
  width: 100%;
  height: 100%;
  height: calc(100vh - 50px);
  position: fixed;
  top: 50px;
  bottom: 0;
}
</style>
