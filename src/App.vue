<script setup>
import { 
  ref, reactive, onMounted, 
  provide, inject, watch,
  isProxy, toRaw } from "vue";
// The URL on your server where CesiumJS's static files are hosted.
window.CESIUM_BASE_URL = '/Cesium';

import { Cartesian3, createOsmBuildingsAsync, Ion, Math as CesiumMath, Terrain, Viewer } from 'cesium';
import "cesium/Build/Cesium/Widgets/widgets.css";

// Your access token can be found at: https://ion.cesium.com/tokens.
// Replace `your_access_token` with your Cesium ion access token.

Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyOTZjNzE5ZS00ZjczLTQ4NGMtOTY1Zi1kZDlhMTU5ZTUzYzkiLCJpZCI6MTQ3NzE0LCJpYXQiOjE3MDMxMzI1OTd9.6BsWfZC1H_THaVpj-CF0DlqqvFCtrojtpsPeMxFnVgU';

onMounted(function () {
  // Initialize the Cesium Viewer in the HTML element with the `cesiumContainer` ID.
  const viewer = new Viewer('cesiumContainer', {
    terrain: Terrain.fromWorldTerrain(),
  });    

  // Fly the camera to San Francisco at the given longitude, latitude, and height.
  viewer.camera.flyTo({
    destination: Cartesian3.fromDegrees(-122.4175, 37.655, 400),
    orientation: {
      heading: CesiumMath.toRadians(0.0),
      pitch: CesiumMath.toRadians(-15.0),
    }
  });

  // Add Cesium OSM Buildings, a global 3D buildings layer.
  createOsmBuildingsAsync().then((buildingTileset)=>{
    viewer.scene.primitives.add(buildingTileset);   
  })
  
});

</script>

<template>
  <div id="cesiumContainer"></div>
</template>

<style>
html, body{
  width: 100%;
  height: 100%;
}
#app{
  position: relative;
  height: 100%;
}

#cesiumContainer{
  position: relative;
  height: 100%;
}

</style>
