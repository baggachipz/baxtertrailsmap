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
      <l-marker v-if="markerLatLng" :lat-lng="markerLatLng">
        <l-icon class="marker-icon">
          <div class="location-marker"></div>
        </l-icon>
      </l-marker>
      <l-marker v-if="markerLatLng && heading" :lat-lng="markerLatLng">
        <l-icon class="direction-icon">
          <div class="direction-marker">
            <q-img
              height="100px"
              src="~assets/direction.svg"
              class="direction-marker-image"
              :style="getHeadingStyle"
            ></q-img>
          </div>
        </l-icon>
      </l-marker>
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
  LIcon,
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
    LIcon,
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
      if (l.heading) {
        this.heading = Math.round(l.heading);
      }

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
    getHeadingStyle() {
      return this.heading ? `transform: rotate(${this.heading}deg)` : null;
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
      heading: null,
    };
  },
};
</script>
<style lang="scss">
body {
  position: fixed;
  height: 100%;
}

.leaflet-div-icon {
  border: 0;
  background: none;
}
.map-container {
  width: 100%;
  height: 100%;
  height: calc(100vh - 50px);
  position: fixed;
  top: 50px;
  bottom: 0;
}

.direction-marker {
  position: absolute;
  opacity: 0.7;
  left: 7px;
  top: -1.5px;
  transform: translateX(-50%) translateY(-50%);
  width: 30px;
  height: 100px;

  .direction-marker-image {
    margin-left: -1.5px;
    margin-top: 6.5px;
  }
}

.location-marker {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
  width: 15px;
  height: 15px;

  &:before {
    content: "";
    position: relative;
    display: block;
    width: 300%;
    height: 300%;
    box-sizing: border-box;
    margin-left: -100%;
    margin-top: -100%;
    border-radius: 45px;
    background-color: teal;
    animation: pulse-ring 1.25s cubic-bezier(0.215, 0.61, 0.355, 1) infinite;
  }

  &:after {
    content: "";
    position: absolute;
    left: 0;
    top: 0;
    display: block;
    width: 100%;
    height: 100%;
    background-color: #01a4e9;
    border-radius: 15px;
    box-shadow: 0 0 8px rgba(0, 0, 0, 0.3);
  }
}

@keyframes pulse-ring {
  0% {
    transform: scale(0.33);
  }
  80%,
  100% {
    opacity: 0;
  }
}
</style>
