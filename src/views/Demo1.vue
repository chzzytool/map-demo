<template>
  <div id="container">
    <div id="map"></div>
    <div class="search-box">
      <input v-model="searchKey" @change="handleSearchChange" class="search" />
    </div>
    <div class="layer-box">
      <div class="layer">
        <span class="layer-title">地表覆盖类型</span>
        <div class="radio-item">
          <input
            type="radio"
            class="radio-circle"
            v-model="layerTypeValue"
            name="layerType"
            value="1"
          />
          <span class="radio-label">水体 </span>
        </div>

        <div class="radio-item">
          <input
            type="radio"
            class="radio-circle"
            v-model="layerTypeValue"
            name="layerType"
            value="2"
          />
          <span class="radio-label">湿地</span>
        </div>

        <div class="radio-item">
          <input
            type="radio"
            class="radio-circle"
            v-model="layerTypeValue"
            name="layerType"
            value="3"
          />
          <span class="radio-label">耕地</span>
        </div>

        <div class="radio-item">
          <input
            type="radio"
            class="radio-circle"
            v-model="layerTypeValue"
            name="layerType"
            value="4"
          />
          <span class="radio-label">森林</span>
        </div>

        <div class="radio-item">
          <input
            type="radio"
            class="radio-circle"
            v-model="layerTypeValue"
            name="layerType"
            value="5"
          />
          <span class="radio-label">草地</span>
        </div>
      </div>
    </div>
    <!-- <div ref="info" class="info-box" v-if="info.name"> -->
    <div ref="info" class="info-box">
      <span class="info1">省名称：{{ info.name }} </span>
      <span class="info1">搜索：{{ searchKey }} </span>
      <span class="info1">图层：{{ layerTypeValue }} </span>
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
      searchKey: "", //搜索关键字
      layerTypeValue: "", //图层选择值
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
          // url: "https://geo.datav.aliyun.com/areas_v2/bound/100000_full.json",
          url: "https://geo.datav.aliyun.com/areas_v2/bound/100000.json",
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
        var pixel = map.getEventPixel(evt.originalEvent);
        map.forEachFeatureAtPixel(pixel, function (feature) {
          if (feature != undefined) {
            console.log(feature);
          }
        });
      });

      map.on("singleclick", function (e) {
        console.log("e->", e);
        const arr1 = olProj.transform(e.coordinate, "EPSG:3857", "EPSG:4326");
        const arr2 = olProj.transform(
          map.getEventCoordinate(e.originalEvent),
          "EPSG:3857",
          "EPSG:4326"
        );
        const arr3 = olProj.transform(
          map.getCoordinateFromPixel(e.pixel),
          "EPSG:3857",
          "EPSG:4326"
        );
        console.log("经纬度1-->", arr1);
        console.log("经纬度2-->", arr2);
        console.log("经纬度3-->", arr3);
      });
    },

    handleSearchChange(e) {
      const value = e.target.value;
      console.log("value->", value);
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
.search {
  padding: 4px 4px 5px 6px;
  font-size: 14px;
  border-radius: 2px;
  border: 1px solid #ccc;
  outline: 0;
}
.search:focus {
  border: 1px solid #aaa;
}
.layer-box {
  position: absolute;
  right: 12px;
  top: 12px;
  width: 120px;
  background: #d3d7d4;
  font-size: 14px;
  border-radius: 2px;
}
.layer {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.layer-title {
  text-align: left;
  padding-left: 6px;
}

.radio-item {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
}

.radio-circle {
  font-size: 14px;
  /* width: 20px;
  height: 20px; */
  padding: 0;
  background-color: #fff;
  border: 1px solid #c9c9c9;
  border-radius: 50%;
  outline: none;
  margin-right: 22px;
  cursor: pointer;
  background-color: indianred;
}

.radio-circle:checked {
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
