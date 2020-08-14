<template>

  <div>
    <l-map
      :zoom.sync="zoom"
      :options="mapOptions"
      
      :bounds="bounds"
      :min-zoom="minZoom"
      :max-zoom="maxZoom"
      style="height: 500px; width: 100%"
    >
      <l-control-layers
        :position="layersPosition"
        :collapsed="false"
        :sort-layers="true"
      />
      <l-tile-layer
        v-for="tileProvider in tileProviders"
        :key="tileProvider.name"
        :name="tileProvider.name"
        :visible="tileProvider.visible"
        :url="tileProvider.url"
        :attribution="tileProvider.attribution"
        :token="token"
        layer-type="base"
      />
      <l-control-zoom :position="zoomPosition" />
      <l-control-attribution
        :position="attributionPosition"
        :prefix="attributionPrefix"
      />
      <l-control-scale :imperial="imperial" />
      <l-marker
        v-for="marker in markers"
        :key="marker.id"
        :visible="marker.visible"
        :draggable="marker.draggable"
        :lat-lng.sync="marker.position"
        :icon="marker.icon"
        @click="alert(marker)"
      >
        <l-popup :content="marker.tooltip" />
        <l-tooltip :content="marker.tooltip" />
      </l-marker>
    </l-map>

  </div>
</template>

<script>
 import { latLngBounds } from 'leaflet';
import {
  LMap,
  LTileLayer,
  LMarker,
  LTooltip,
  LPopup,
  LControlZoom,
  LControlAttribution,
  LControlScale,
  LControlLayers
} from 'vue2-leaflet';

const axios = require('axios');
const moment = require('moment');

const tileProviders = [
  {
    name: 'OpenStreetMap',
    visible: true,
    attribution:
      '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
    url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
  },
  {
    name: 'OpenTopoMap',
    visible: false,
    url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
    attribution:
      'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
  },
];

export default {
  name: 'Example',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LPopup,
    LControlZoom,
    LControlAttribution,
    LControlScale,
    LControlLayers,
  },
  mounted: function(){
    this.getMarkers();
  },
  data() {
    return {
      // center: [51.505, -0.09],
      opacity: 0.6,
      token: 'your token if using mapbox',
      mapOptions: {
        zoomControl: false,
        attributionControl: false,
        zoomSnap: true,
      },
      zoom: 3,
      minZoom: 1,
      maxZoom: 20,
      zoomPosition: 'topleft',
      attributionPosition: 'bottomright',
      layersPosition: 'topright',
      attributionPrefix: 'Vue2Leaflet',
      imperial: false,      
      tileProviders: tileProviders,
      markers: [],
      bounds: null
    };
  },
  methods: {
    alert(item) {
      alert('this is ' + JSON.stringify(item));
    },
    getMarkers: function(){
      var url = process.env.VUE_APP_API;
      var day = moment().format('YYYY-MM-DD');  //todo as arg      
      var m = [];
      axios      
      .get(url + '?day=' + day)
      .then(function(response){              
        response.data.forEach(function(pos, k){
            m.push({
              id: pos.origin + '_' + k,
              tooltip: "<ul><li>date: " + pos.at + "</li><li>" + "alt: " + pos.alt + "</li><li>" + "volts: " + pos.batt + "</li></ul>",
              position: { "lat": pos.lat, "lng": pos.lon },

            })
        });  
      });

      this.markers = m;                 
      this.bounds =  latLngBounds([
        [43.70081290280357, 5.26963806152345],
        [45.82991732677597, 5.08716201782228]
      ]);
    }    
  },
};
</script>
