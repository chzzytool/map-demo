<template>
  <div id="map-container"></div>
</template>

<script>
import "ol/ol.css";
import { Map, View } from "ol";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
export default {
  mounted() {
    new Map({
      target: "map-container",
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
      ],
      view: new View({
        center: [0, 0],
        zoom: 0,
      }),
    });
  },
};
</script>

<style scoped>
#map-container {
  width: 100%;
  height: 100%;
}
</style>
