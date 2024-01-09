<script setup>
import { 
  ref, reactive, onMounted, 
  provide, inject, watch,
  isProxy, toRaw } from "vue";


import { 
  Cartesian3, 
  createOsmBuildingsAsync, 
  Cesium3DTileset, 
  Ion, Math as CesiumMath, 
  Terrain, Viewer,
  RequestScheduler } from 'cesium';
import "cesium/Build/Cesium/Widgets/widgets.css";
import {
  MDBInput,  MDBTextarea,
  MDBCol,  MDBRow,  MDBContainer,
  MDBSelect,  MDBDatepicker,  MDBBtn,  MDBPopconfirm,
  MDBSpinner,  MDBAnimation,  MDBAlert,
  MDBModal,  MDBModalHeader,  MDBModalTitle,  MDBModalBody,  MDBModalFooter,
  MDBSwitch,
} from 'mdb-vue-ui-kit';

//#region ======參數======
  // The URL on your server where CesiumJS's static files are hosted.
  window.CESIUM_BASE_URL = '/Cesium';
  const positionWC_str = ref('');
  const positionWC_step = ref(1000);

  const positionCartographic_str = ref('');
  const Oritation_str = ref('');
  const directionWC_str = ref('');

  // Your access token can be found at: https://ion.cesium.com/tokens.
  // Replace `your_access_token` with your Cesium ion access token.
  Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyOTZjNzE5ZS00ZjczLTQ4NGMtOTY1Zi1kZDlhMTU5ZTUzYzkiLCJpZCI6MTQ3NzE0LCJpYXQiOjE3MDMxMzI1OTd9.6BsWfZC1H_THaVpj-CF0DlqqvFCtrojtpsPeMxFnVgU';

//#endregion ======參數======

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
  camera.flyTo({
    destination: Cartesian3.fromDegrees(120.6638, 24.147, 2000),
    orientation: {
      heading: CesiumMath.toRadians(0.0),
      pitch: CesiumMath.toRadians(-85.0),
    }
  });
  
  showInfo(camera);
  camera.changed.addEventListener(()=>{
    showInfo(camera);
  })
  // Add Cesium OSM Buildings, a global 3D buildings layer.
  // createOsmBuildingsAsync().then((buildingTileset)=>{
  //   viewer.scene.primitives.add(buildingTileset);   
  // })
  // Cesium3DTileset.fromUrl("https://tile.googleapis.com/v1/3dtiles/root.json?key=AIzaSyD_BVTqEVIUJJ6rqbY571GsDemijM7RZFc").then(tileset=>{
  //   viewer.scene.primitives.add(tileset); 
  // });
});

function showInfo(camera){
  // positionWC
  positionWC_str.value = camera.positionWC.toString()
  // positionCartographic
  positionCartographic_str.value = 
    '(' + CesiumMath.toDegrees(camera.positionCartographic.latitude) + ',' + 
    CesiumMath.toDegrees(camera.positionCartographic.longitude) + ',' +
    camera.positionCartographic.height + ')';

  // Oritation
  Oritation_str.value =
    '(' + CesiumMath.toDegrees(camera.roll) + ',' + 
    CesiumMath.toDegrees(camera.pitch) + ',' +
    CesiumMath.toDegrees(camera.heading) + ')';

  // directionWC
  directionWC_str.value = camera.directionWC.toString();
}

</script>

<template>
  <div id="cesiumContainer"></div>
  <MDBContainer id="vinfo">
    <!-- positionWC -->
    <MDBRow>
      <MDBCol col="2">positionWC</MDBCol>
      <MDBCol col="7">{{ positionWC_str }}</MDBCol>
      <MDBCol col="3" class="p-1">
        <MDBRow>
          <!-- step -->
          <MDBCol col="6" class="d-flex align-items-center">
            <MDBInput size="sm" v-model="positionWC_step"></MDBInput>
          </MDBCol>
          <MDBCol v-for="n in 3" class="d-flex flex-column align-items-start p-0">
            <MDBBtn color="primary" class="btn-updowm" @click="setValueBtn(n,1)">+</MDBBtn>
            <MDBBtn color="primary" class="btn-updowm mt-1" @click="setValueBtn(n,-1)">-</MDBBtn>
          </MDBCol>
        </MDBRow>
      </MDBCol>
    </MDBRow>
    <!-- positionCartographic -->
    <MDBRow>
      <MDBCol col="2">positionCartographic</MDBCol>
      <MDBCol col="7">{{ positionCartographic_str }}</MDBCol>
      <MDBCol col="3">tools</MDBCol>
    </MDBRow>
    <!-- Oritation -->
    <MDBRow>
      <MDBCol col="2">Oritation</MDBCol>
      <MDBCol col="7">{{ Oritation_str }}</MDBCol>
      <MDBCol col="3">tools</MDBCol>
    </MDBRow>
    <!-- directionWC -->
    <MDBRow>
      <MDBCol col="2">directionWC</MDBCol>
      <MDBCol col="7">{{ directionWC_str }}</MDBCol>
      <MDBCol col="3">tools</MDBCol>
    </MDBRow>
  </MDBContainer>
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
  font-family: Roboto, Helvetica, Arial, sans-serif;
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

.btn-updowm{
  padding: 0!important;
  width: 1.5rem;
  height: 1rem;
}
</style>
