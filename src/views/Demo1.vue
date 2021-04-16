<template>
  <div id="container">
    <div id="map"></div>
    <div class="search-box">
      <input class="search" />
    </div>
    <!-- <div class="layer-box">
      <span class="layer-title">地表覆盖类型</span>
      <input type="radio" name="layerType" value="1" />水体
      <input type="radio" name="layerType" value="2" />湿地
      <input type="radio" name="layerType" value="3" />耕地
      <input type="radio" name="layerType" value="4" />森林
      <input type="radio" name="layerType" value="5" />草地
    </div> -->
    <div ref="info" class="info-box" v-if="info.name">
      <span class="info1">省名称：{{ info.name }} </span>
    </div>
  </div>
</template>

<script>
import "ol/ol.css";
import GeoJSON from "ol/format/GeoJSON";
import Map from "ol/Map";
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import View from "ol/View";
import { Fill, Stroke, Style, Text } from "ol/style";
import * as olProj from "ol/proj";

export default {
  data() {
    return {
      info: {
        name: "",
      }, //描述信息
    };
  },
  methods: {
    init() {
      const that = this;
      const style = new Style({
        fill: new Fill({
          color: "rgba(255, 255, 255, 0.6)",
        }),
        stroke: new Stroke({
          color: "#319FD3",
          width: 1,
        }),
        text: new Text({
          font: "12px Calibri,sans-serif",
          fill: new Fill({
            color: "#000",
          }),
          stroke: new Stroke({
            color: "#fff",
            width: 3,
          }),
        }),
      });

      const vectorLayer = new VectorLayer({
        source: new VectorSource({
          url: "https://geo.datav.aliyun.com/areas_v2/bound/100000_full.json",
          format: new GeoJSON(),
        }),
        style: function (feature) {
          style.getText().setText(feature.get("name"));
          return style;
        },
      });

      const map = new Map({
        layers: [vectorLayer],
        target: "map",
        view: new View({
          //经纬度坐标(4326)转换成墨卡托(3857)
          center: olProj.transform([104, 30], "EPSG:4326", "EPSG:3857"),
          zoom: 4,
        }),
      });

      const highlightStyle = new Style({
        stroke: new Stroke({
          color: "#f00",
          width: 1,
        }),
        fill: new Fill({
          color: "rgba(255,0,0,0.1)",
        }),
        text: new Text({
          font: "12px Calibri,sans-serif",
          fill: new Fill({
            color: "#000",
          }),
          stroke: new Stroke({
            color: "#f00",
            width: 3,
          }),
        }),
      });

      const featureOverlay = new VectorLayer({
        source: new VectorSource(),
        map: map,
        style: function (feature) {
          highlightStyle.getText().setText(feature.get("name"));
          return highlightStyle;
        },
      });

      let highlight;
      const displayFeatureInfo = function (pixel) {
        const feature = map.forEachFeatureAtPixel(pixel, function (feature) {
          return feature;
        });

        // const info = that.$refs.info;
        if (feature) {
          // info.innerHTML = feature.getId() + ": " + feature.get("name");
          that.info.name = feature.get("name");
          console.log("feature-->", feature.get("name"));
        } else {
          that.info.name = "";
        }

        if (feature !== highlight) {
          if (highlight) {
            featureOverlay.getSource().removeFeature(highlight);
          }
          if (feature) {
            featureOverlay.getSource().addFeature(feature);
          }
          highlight = feature;
        }
      };

      map.on("pointermove", function (evt) {
        if (evt.dragging) {
          return;
        }
        const pixel = map.getEventPixel(evt.originalEvent);
        displayFeatureInfo(pixel);
      });

      map.on("click", function (evt) {
        displayFeatureInfo(evt.pixel);
        console.log("evt-->", evt);
      });

      map.on("singleclick", function (e) {
        console.log("e->", e);
        console.log(
          "经纬度1-->",
          olProj.transform(e.coordinate, "EPSG:3857", "EPSG:4326")
        );
        console.log(
          "经纬度2-->",
          olProj.transform(
            map.getEventCoordinate(e.originalEvent),
            "EPSG:3857",
            "EPSG:4326"
          )
        );
        console.log(
          "经纬度3-->",
          olProj.transform(
            map.getCoordinateFromPixel(e.pixel),
            "EPSG:3857",
            "EPSG:4326"
          )
        );
      });
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style scoped>
#container {
  width: 100%;
  height: 100%;
}
#map {
  width: 100%;
  height: 100%;
}
.search-box {
  position: absolute;
  left: 100px;
  top: 12px;
}
.layer-box {
  position: absolute;
  right: 100px;
  top: 12px;
}
.info-box {
  position: absolute;
  right: 12px;
  bottom: 10px;
  padding: 10px;
  border-radius: 2px;
  background: #d3d7d4;
}
</style>
