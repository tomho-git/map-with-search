<template>
  <div id="mapContainer" ref="mapContainer"></div>
</template>

<script>
import Eventbus from "../tools/eventbus";

import "leaflet/dist/leaflet.css";
import L from "leaflet";
import { Icon } from "leaflet";

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
  iconUrl: require("leaflet/dist/images/marker-icon.png"),
  shadowUrl: require("leaflet/dist/images/marker-shadow.png"),
});
export default {
  name: "mapToShow",
  data: function () {
    return {
      mapObject: null,
      center: [43.776117, -79.419727], // north york lat long
      currentLocationMarker: null,
      markerList: [],
    };
  },
  components: {},
  methods: {
    setupLeafletMap: function () {
      this.mapObject = L.map("mapContainer").setView(this.center, 13);
      L.tileLayer("http://{s}.google.com/vt?lyrs=s,h&x={x}&y={y}&z={z}", {
        maxZoom: 20,
        subdomains: ["mt0", "mt1", "mt2", "mt3"],
      }).addTo(this.mapObject);
    },
    removeAllMarkers: function () {
      for (var key in this.markerList) {
        var obj = this.markerList[key];
        this.mapObject.removeLayer(obj);
      }
      this.markerList = [];
    },
    addMarkers: function (list) {
      this.removeAllMarkers();
      for (var i = 0; i < list.length; i++) {
        this.addMarker(list[i].lat,list[i].lon, list[i].place_id);
      }
    },
    addMarker: function (lat, long, id) {
      if ((lat, long, id)) {
        var marker = L.marker([lat, long]);
        var temp = marker.addTo(this.mapObject);
        this.markerList[id] = temp;
      }
    },
    currentLocation: function (position) {
      if (
        position &&
        position.coords &&
        position.coords.latitude &&
        position.coords.longitude
      ) {
        this.addCurrentPoint(
          position.coords.latitude,
          position.coords.longitude
        );
        this.flyTo(position.coords.latitude, position.coords.longitude, 16);
      }
    },
    flyTo(lat, long, zoom) {
      this.mapObject.flyTo([lat, long], zoom);
    },
    addCurrentPoint(lat, long) {
      if (this.currentLocationMarker) {
        this.mapObject.removeLayer(this.currentLocationMarker);
      }
      var greenIcon = L.icon({
        iconUrl: require("../assets/currentLocation.svg"),
        iconSize: [38, 95], // size of the icon
        iconAnchor: [22, 94], // point of the icon which will correspond to marker's location
        shadowAnchor: [4, 62], // the same for the shadow
        popupAnchor: [-3, -76], // point from which the popup should open relative to the iconAnchor
      });
      this.currentLocationMarker = L.marker([lat, long], {
        icon: greenIcon,
      }).addTo(this.mapObject);
    },
  },
  mounted() {
    this.setupLeafletMap();
    Eventbus.$on("addMarkers", this.addMarkers);
    Eventbus.$on("currentLocation", this.currentLocation);
  },
  destroyed() {
    Eventbus.$off("addMarkers");
    Eventbus.$off("currentLocation");
  },
};
</script>

<style scoped>
#mapContainer {
  height: 100%;
}
</style>
