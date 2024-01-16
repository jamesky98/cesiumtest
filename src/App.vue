<script setup>
import { 
  ref, reactive, onMounted, 
  provide, inject, watch,
  isProxy, toRaw } from "vue";


import { 
  Cartesian3, 
  Cesium3DTileStyle,
  createOsmBuildingsAsync, 
  Cesium3DTileset, 
  Ion, Math as CesiumMath, 
  Terrain, Viewer,
  RequestScheduler,
  viewerCesium3DTilesInspectorMixin } from 'cesium';
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
  const positionWC_str = ref([]);
  const positionWC_step = ref(1000);

  const positionCartographic_str = ref([]);
  const Oritation_str = ref([]);
  const directionWC_str = ref([]);
  const upWC_str = ref([]);

  const moveAmount = ref(10000);
  const forwardAmount = ref(1000000);
  const angleAmount = ref(10);

  let cs_viewer;
  let cs_camera;

  // Your access token can be found at: https://ion.cesium.com/tokens.
  // Replace `your_access_token` with your Cesium ion access token.
  Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyOTZjNzE5ZS00ZjczLTQ4NGMtOTY1Zi1kZDlhMTU5ZTUzYzkiLCJpZCI6MTQ3NzE0LCJpYXQiOjE3MDMxMzI1OTd9.6BsWfZC1H_THaVpj-CF0DlqqvFCtrojtpsPeMxFnVgU';

//#endregion ======參數======

// google Maptile 加速使用
RequestScheduler.requestsByServer["tile.googleapis.com:443"] = 18;

onMounted(function () {
  // Initialize the Cesium Viewer in the HTML element with the `cesiumContainer` ID.
  cs_viewer = new Viewer('cesiumContainer', {
    animation: false,
    baseLayerPicker: false,
    // fullscreenButton: false,
    // vrButton: false,
    geocoder: false,
    // homeButton: false,
    // infoBox: false,
    sceneModePicker: false,
    // selectionIndicator: false,
    timeline: true,
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
    // terrain: Terrain.fromWorldTerrain(),
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
    shadows: false,
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
  cs_viewer.bottomContainer.style.display = "none";
  cs_camera = cs_viewer.camera;

  // cs_viewer.extend(viewerCesium3DTilesInspectorMixin);
  // const inspectorViewModel = cs_viewer.cesium3DTilesInspector.viewModel;

  cs_viewer.scene.debugShowFramesPerSecond = true;
  // cs_camera.upWC.x
  // Fly the camera to San Francisco at the given longitude, latitude, and height.
  // cs_camera.flyTo({
  //   // destination: Cartesian3.fromDegrees(120.6638, 24.147, 2000),
  //   destination: Cartesian3.fromDegrees(121,24,100000),
  //   // orientation: {
  //   //   heading: CesiumMath.toRadians(0.0),
  //   //   pitch: CesiumMath.toRadians(-85.0),
  //   // }
  // });
  
  showInfo(cs_camera);
  cs_camera.changed.addEventListener(()=>{
    showInfo(cs_camera);
  })
  cs_camera.moveEnd.addEventListener(()=>{
    showInfo(cs_camera);
  })
  // Add Cesium OSM Buildings, a global 3D buildings layer.
  // createOsmBuildingsAsync().then((buildingTileset)=>{
  //   cs_viewer.scene.primitives.add(buildingTileset);   
  // })
  // Cesium3DTileset.fromUrl("https://tile.googleapis.com/v1/3dtiles/root.json?key=AIzaSyD_BVTqEVIUJJ6rqbY571GsDemijM7RZFc").then(tileset=>{
  //   viewer.scene.primitives.add(tileset); 
  // });
  // 自動遮蔽低於地形高度的模型
  cs_viewer.scene.globe.depthTestAgainstTerrain = true;
  Cesium3DTileset.fromUrl("/samples/utrecht3d/tileset.json").then(tileset=>{

    // tileset.style = new Cesium3DTileStyle({
    //   color: {
    //     conditions: [
    //       ["${CLASSIFICATION} === 2", "color('brown')"],       // ground
    //       ["${CLASSIFICATION} === 3", "color('greenyellow')"], // low vegetation
    //       ["${CLASSIFICATION} === 4", "color('green')"],       // medium vegetation
    //       ["${CLASSIFICATION} === 5", "color('darkgreen')"],   // high vegetation
    //       ["${CLASSIFICATION} === 6", "color('red')"],   // high vegetation
    //     ]
    //   },
    //   pointSize: 4
    // });


    cs_viewer.scene.primitives.add(tileset);
    cs_viewer.zoomTo(tileset);
    // cs_camera.flyTo({
    //   destination: Cartesian3.fromDegrees(121,24,100000),
    // })
  });
});

function showInfo(camera){
  // positionWC
  // positionWC_str.value = camera.positionWC.toString();
  positionWC_str.value = [
    camera.positionWC.x.toFixed(3),
    camera.positionWC.y.toFixed(3),
    camera.positionWC.z.toFixed(3)
  ]
  // positionCartographic
  positionCartographic_str.value = [
    CesiumMath.toDegrees(camera.positionCartographic.longitude).toFixed(8),
    CesiumMath.toDegrees(camera.positionCartographic.latitude).toFixed(8),
    camera.positionCartographic.height.toFixed(3)
  ]
  // Oritation
  Oritation_str.value = [
    CesiumMath.toDegrees(camera.roll).toFixed(8),
    CesiumMath.toDegrees(camera.pitch).toFixed(8),
    CesiumMath.toDegrees(camera.heading).toFixed(8),
  ];

  // directionWC
  directionWC_str.value = [
    camera.directionWC.x.toFixed(3),
    camera.directionWC.y.toFixed(3),
    camera.directionWC.z.toFixed(3)
  ];

  // upWC
  upWC_str.value = [
    camera.upWC.x.toFixed(3),
    camera.upWC.y.toFixed(3),
    camera.upWC.z.toFixed(3)
  ];
}

function moveCamera(camera,action){
  if(action==='moveUp'){
    camera.moveUp(parseFloat(moveAmount.value));
  }else if(action==='moveDown'){
    camera.moveDown(parseFloat(moveAmount.value));
  }else if(action==='moveLeft'){
    camera.moveLeft(parseFloat(moveAmount.value));
  }else if(action==='moveRight'){
    camera.moveRight(parseFloat(moveAmount.value));
  }else if(action==='moveForward'){
    camera.moveForward(parseFloat(forwardAmount.value));
  }else if(action==='moveBackward'){
    camera.moveBackward(parseFloat(forwardAmount.value));
  }else if(action==='lookUp'){
    camera.lookUp(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='lookDown'){
    camera.lookDown(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='lookLeft'){
    camera.lookLeft(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='lookRight'){
    camera.lookRight(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='zoomIn'){
    camera.zoomIn(parseFloat(forwardAmount.value));
  }else if(action==='zoomOut'){
    camera.zoomOut(parseFloat(forwardAmount.value));
  }else if(action==='twistLeft'){
    camera.twistLeft(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='twistRight'){
    camera.twistRight(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='rotateUp'){
    camera.rotateUp(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='rotateDown'){
    camera.rotateDown(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='rotateLeft'){
    camera.rotateLeft(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='rotateRight'){
    camera.rotateRight(CesiumMath.toRadians(parseFloat(angleAmount.value)));
  }else if(action==='flyhome'){
    camera.flyTo({
      destination: Cartesian3.fromDegrees(121,24,100000)
    })
  }
}

</script>

<template>
  <MDBContainer fluid class="h-100">
    <div id="cesiumContainer" class="border">
      <div class="cursorH border-bottom border-danger"></div>
      <div class="cursorV border-end border-danger"></div>
    </div>
    <!-- positionWC -->
    <MDBRow id="vinfo">
      <!-- block 1 -->
      <MDBCol>
        <MDBRow class="text-start">
          <MDBCol col="2" class="overflow-hidden">positionWC</MDBCol>
          <MDBCol>{{ positionWC_str[0] }}</MDBCol>
          <MDBCol>{{ positionWC_str[1] }}</MDBCol>
          <MDBCol>{{ positionWC_str[2] }}</MDBCol>
          <div></div>
          <MDBCol col="2" class="overflow-hidden">positionCartographic</MDBCol>
          <MDBCol>{{ positionCartographic_str[0] }}</MDBCol>
          <MDBCol>{{ positionCartographic_str[1] }}</MDBCol>
          <MDBCol>{{ positionCartographic_str[2] }}</MDBCol>
          <div></div>
          <MDBCol col="2" class="overflow-hidden">Oritation</MDBCol>
          <MDBCol>{{ Oritation_str[0] }}</MDBCol>
          <MDBCol>{{ Oritation_str[1] }}</MDBCol>
          <MDBCol>{{ Oritation_str[2] }}</MDBCol>
          <div></div>
          <MDBCol col="2" class="overflow-hidden">directionWC</MDBCol>
          <MDBCol>{{ directionWC_str[0] }}</MDBCol>
          <MDBCol>{{ directionWC_str[1] }}</MDBCol>
          <MDBCol>{{ directionWC_str[2] }}</MDBCol>
          <div></div>
          <MDBCol col="2" class="overflow-hidden">upWC</MDBCol>
          <MDBCol>{{ upWC_str[0] }}</MDBCol>
          <MDBCol>{{ upWC_str[1] }}</MDBCol>
          <MDBCol>{{ upWC_str[2] }}</MDBCol>
        </MDBRow>
      </MDBCol>
      <!-- block 2 -->
      <MDBCol col="2">
        <MDBRow>
          <MDBCol col="12">
            <MDBInput v-model="moveAmount"></MDBInput>
          </MDBCol>
          <MDBCol v-for="x in ['moveUp','moveDown','moveLeft','moveRight']" col="6">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,x)">{{ x }}</MDBBtn>
            </MDBRow>
          </MDBCol>
          <MDBCol col="12">
            <MDBInput v-model="forwardAmount"></MDBInput>
          </MDBCol>
          <MDBCol v-for="x in ['moveForward','moveBackward']" col="6">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,x)">{{ x }}</MDBBtn>
            </MDBRow>
          </MDBCol>
        </MDBRow>
      </MDBCol>
      <!-- block 3 -->
      <MDBCol col="1">
        <MDBRow>
          <MDBCol col="12">
            <MDBInput v-model="angleAmount"></MDBInput>
          </MDBCol>
          <MDBCol v-for="x in ['lookUp','lookDown','lookLeft','lookRight']" col="12">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,x)">{{ x }}</MDBBtn>
            </MDBRow>
          </MDBCol>
        </MDBRow>
      </MDBCol>
      <!-- block 4 -->
      <MDBCol col="1">
        <MDBRow>
          <MDBCol v-for="x in ['zoomIn','zoomOut','twistLeft','twistRight']" col="12">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,x)">{{ x }}</MDBBtn>
            </MDBRow>
          </MDBCol>
        </MDBRow>
      </MDBCol>
      <!-- block 5 -->
      <MDBCol col="1">
        <MDBRow>
          <MDBCol v-for="x in ['rotateUp','rotateDown','rotateLeft','rotateRight']" col="12">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,x)">{{ x }}</MDBBtn>
            </MDBRow>
          </MDBCol>
          <MDBCol col="12">
            <MDBRow class="p-1">
              <MDBBtn color="primary" class="text-truncate" @click="moveCamera(cs_camera,'flyhome')">home</MDBBtn>
            </MDBRow>
          </MDBCol>
        </MDBRow>
      </MDBCol>
    </MDBRow>
  </MDBContainer>
</template>

<style>
:root {
  --bottom-div: 12rem;
}

html, body{
  width: 100%;
  height: 100%;
}
#app{
  font-family: Roboto, Helvetica, Arial, sans-serif;
  height: 100%;
  width: 100%;
  margin: 0;
  max-width: 100%;
}

#cesiumContainer{
  position: relative;
  height: calc(100% - var(--bottom-div));
}
#vinfo{
  position: relative;
  height: var(--bottom-div);
  width: 100%;
}

.cursorH{
  position: absolute;
  left: 0;
  top: 50%;
  width: 100%;
  height: 0;
  z-index: 100;
}

.cursorV{
  position: absolute;
  left: 50%;
  top: 0;
  width: 0;
  height: 100%;
  z-index: 100;
}

</style>
