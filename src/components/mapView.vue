<template>
    <div>
        <div id="map" class="map-settings"></div>
    </div>    
</template>
<script>
import L from "leaflet";
import axios from "axios";
export default {
  data() {
    return {
      map: null,
      mapData: ["78mm.json", "100mm_hr.json", "130mm_hr.json"],
      geoJsonData: [],
      tileLayer: null,
      controlLayer: null,
      overlayMaps: []
    };
  },
  mounted() {
    this.initMap();
    this.getKmlData();
  },
  methods: {
    initMap() {
      this.map = L.map("map", {
        center: [25.0419643, 121.5320798],
        zoom: 13
      });
      this.tileLayer = {
        Basemap: L.tileLayer(
          "https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager_labels_under/{z}/{x}/{y}{r}.png",
          {
            attribution:
              '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> &copy; <a href="http://cartodb.com/attributions">CartoDB</a>',
            subdomains: "abcd",
            maxZoom: 19
          }
        )
      };
      this.tileLayer.Basemap.addTo(this.map);
      this.controlLayer = L.control
        .layers(this.tileLayer, null)
        .addTo(this.map);
    },
    getKmlData() {
      axios
        .all(
          this.mapData.map(fs => {
            return axios.get(fs);
          })
        )
        .then(
          axios.spread((...payload) => {
            return payload;
          })
        )
        .then(res => {
          res.forEach(data => {
            this.geoJsonData.push({
              name: data.config.url.replace(".json", ""),
              geoJson: data.data
            });
          });
        })
        .then(() => {
          this.geoJsonData.forEach(data => {
            this.drawGeoJson(data);
          });
        });
    },
    drawGeoJson(geoJsonFeature) {
      let name = geoJsonFeature.name;
      let layer = null;
      if (name === "78mm") {
        layer = L.geoJson(geoJsonFeature.geoJson, {
          style: this.drawLineStyle75
        }).addTo(this.map);
      } else if (name === "100mm_hr") {
        layer = L.geoJson(geoJsonFeature.geoJson, {
          style: this.drawLineStyle100
        }).addTo(this.map);
      } else {
        layer = L.geoJson(geoJsonFeature.geoJson, {
          style: this.drawLineStyle130
        }).addTo(this.map);
      }
      this.saveToOverlayMap(name, layer);
    },
    saveToOverlayMap(name, layer) {
      //save to overlay variable
      this.overlayMaps.push({
        name: name,
        layer: layer
      });
      //put into map
      this.controlLayer.addOverlay(layer, name);
    },
    drawLineStyle75(geoJsonObject) {
      if (geoJsonObject.properties.name === "0.3")
        return {
          weight: 2,
          color: "yellow"
        };
      else {
        return {
          weight: 3,
          color: "yelllow"
        };
      }
    },
    drawLineStyle100(geoJsonObject) {
      if (geoJsonObject.properties.name === "0.3")
        return {
          weight: 2,
          color: "orange"
        };
      else {
        return {
          weight: 3,
          color: "orange"
        };
      }
    },
    drawLineStyle130(geoJsonObject) {
      if (geoJsonObject.properties.name === "0.3")
        return {
          weight: 2,
          color: "red"
        };
      else {
        return {
          weight: 3,
          color: "red"
        };
      }
    }
  }
};
</script>
<style>
.map-settings {
  height: 600px;
  width: 100%;
}
</style>
