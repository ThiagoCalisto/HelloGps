<template>
  <div>
    <div>
      <label for="type">Geometry type:</label>
      <select id="type" v-model="selectedType">
        <option value="None">None</option>
        <option value="Point">Point</option>
        <option value="LineString">LineString</option>
        <option value="Polygon">Polygon</option>
      </select>
    </div>
    <div>
      <button @click="undo">Undo</button>
    </div>
    <div id="map" style="width: 100%; height: 400px;"></div>
  </div>
</template>

<script>
import 'ol/ol.css';
import Draw from 'ol/interaction/Draw.js';
import Map from 'ol/Map.js';
import View from 'ol/View.js';
import { OSM, Vector as VectorSource } from 'ol/source.js';
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer.js';

export default {
  data() {
    return {
      selectedType: 'None',
      map: null,
      draw: null,
    };
  },
  mounted() {
    this.initMap();
    this.addInteraction();
  },
  methods: {
    initMap() {
      const raster = new TileLayer({
        source: new OSM(),
      });

      const source = new VectorSource({ wrapX: false });

      const vector = new VectorLayer({
        source: source,
      });

      this.map = new Map({
        layers: [raster, vector],
        target: 'map',
        view: new View({
          center: [-11000000, 4600000],
          zoom: 4,
        }),
      });
    },
    addInteraction() {
      if (this.selectedType !== 'None') {
        this.draw = new Draw({
          source: this.map.getLayers().item(1).getSource(),
          type: this.selectedType,
        });
        this.map.addInteraction(this.draw);
      }
    },
    undo() {
      this.draw.removeLastPoint();
    },
  },
  watch: {
    selectedType() {
      this.map.removeInteraction(this.draw);
      this.addInteraction();
    },
  },
};
</script>