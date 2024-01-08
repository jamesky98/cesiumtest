<script setup>
import { 
  ref, reactive, onMounted, 
  provide, inject, watch,
  isProxy, toRaw } from "vue";
// The URL on your server where CesiumJS's static files are hosted.
window.CESIUM_BASE_URL = '/Cesium';

import { 
  Cartesian3, 
  createOsmBuildingsAsync, 
  Cesium3DTileset, 
  Ion, Math as CesiumMath, 
  Terrain, Viewer,
  RequestScheduler } from 'cesium';
import "cesium/Build/Cesium/Widgets/widgets.css";

const vinfo_txt = ref('');
// Your access token can be found at: https://ion.cesium.com/tokens.
// Replace `your_access_token` with your Cesium ion access token.

Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyOTZjNzE5ZS00ZjczLTQ4NGMtOTY1Zi1kZDlhMTU5ZTUzYzkiLCJpZCI6MTQ3NzE0LCJpYXQiOjE3MDMxMzI1OTd9.6BsWfZC1H_THaVpj-CF0DlqqvFCtrojtpsPeMxFnVgU';

// google Maptile 加速使用
RequestScheduler.requestsByServer["tile.googleapis.com:443"] = 18;

onMounted(function () {
  // Initialize the Cesium Viewer in the HTML element with the `cesiumContainer` ID.
  const viewer = new Viewer('cesiumContainer', {
    animation: false,
    baseLayerPicker: false,
    // fullscreenButton: false,
    // vrButton: false,
    geocoder: false,
    // homeButton: false,
    // infoBox: false,
    sceneModePicker: false,
    // selectionIndicator: false,
    timeline: false,
    navigationHelpButton: false,
    // navigationInstructionsInitiallyVisible: true,
    // scene3DOnly: false,
    // shouldAnimate: true,
    // clockViewModel: 
    // selectedImageryProviderViewModel
    // imageryProviderViewModels
    // selectedTerrainProviderViewModel
    // terrainProviderViewModels
    // baseLayer
    // terrainProvider
    terrain: Terrain.fromWorldTerrain(),
    // skyBox
    // skyAtmosphere
    // fullscreenElement
    // useDefaultRenderLoop
    // targetFrameRate
    // showRenderLoopErrors
    // useBrowserRecommendedResolution
    // automaticallyTrackDataSourceClocks
    // contextOptions
    // sceneMode
    // mapProjection
    // globe
    // orderIndependentTranslucency
    // creditContainer
    // creditViewport
    // dataSources
    // shadows: false,
    // terrainShadows
    // mapMode2D
    projectionPicker: false,
    // blurActiveElementOnCanvasFocus: true,
    // requestRenderMode: false,
    // maximumRenderTimeChange
    // depthPlaneEllipsoidOffset
    // msaaSamples
  });   

  // Logo none
  viewer.bottomContainer.style.display = "none";
  const camera = viewer.camera;
  // Fly the camera to San Francisco at the given longitude, latitude, and height.
  // camera.flyTo({
  //   destination: Cartesian3.fromDegrees(-122.4175, 37.655, 400),
  //   orientation: {
  //     heading: CesiumMath.toRadians(0.0),
  //     pitch: CesiumMath.toRadians(-15.0),
  //   }
  // });
  
  vinfo_txt.value = camera.positionWC.toString();
  camera.changed.addEventListener(()=>{
    vinfo_txt.value = camera.positionWC.toString() + '\n';
    // vinfo_txt.value += camera.position.toString() + '\n';
    vinfo_txt.value += 
      '(' + CesiumMath.toDegrees(camera.positionCartographic.latitude) + ',' + 
      CesiumMath.toDegrees(camera.positionCartographic.longitude) + ',' +
      camera.positionCartographic.height + ')\n';
    
    vinfo_txt.value += 
      '(' + CesiumMath.toDegrees(camera.roll) + ',' + 
      CesiumMath.toDegrees(camera.pitch) + ',' +
      CesiumMath.toDegrees(camera.heading) + ')\n';
    
    vinfo_txt.value += camera.directionWC.toString() + '\n';
    vinfo_txt.value += camera.directionWC.toString();
  })
  // Add Cesium OSM Buildings, a global 3D buildings layer.
  createOsmBuildingsAsync().then((buildingTileset)=>{
    viewer.scene.primitives.add(buildingTileset);   
  })
  // Cesium3DTileset.fromUrl("https://tile.googleapis.com/v1/3dtiles/root.json?key=AIzaSyD_BVTqEVIUJJ6rqbY571GsDemijM7RZFc").then(tileset=>{
  //   viewer.scene.primitives.add(tileset); 
  // });
});


</script>

<template>
  <div id="cesiumContainer"></div>
  <div id="vinfo" class="pre-formatted">{{vinfo_txt}}</div>
</template>

<style>
:root {
  --bottom-div: 10rem;
}

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
  height: calc(100% - var(--bottom-div));
}
#vinfo{
  position: relative;
  height: var(--bottom-div);
}

.pre-formatted {
  white-space: pre-line;
}
</style>
