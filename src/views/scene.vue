<template>
  <div id="cesiumContainer"></div>

  <el-select class="changeMap" v-model="currentMap" placeholder="Select" size="small" @change="changeMap">
    <el-option v-for="item in mapOptions" :key="item.value" :label="item.label" :value="item.value"
      :disabled="item.disabled" />
  </el-select>

  <el-button class="loadModel" size="small" @click="loadGLB">加载模型</el-button>
</template>

<script setup>
import { onMounted, ref } from 'vue'
Cesium.CESIUM_BASE_URL = '/Cesium'
Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIwYzA4NzQ3ZC0yOGU0LTRjY2EtYWQwMy1mNGNkODExMTNlNDYiLCJpZCI6MjAyNjY1LCJpYXQiOjE3MTA3NTYxNzR9.GNuJOlYuB5KL_5HPkQEmqMuEBnhG7N21QLUP-nVmeoA';

const currentMap = ref('bing')
const mapOptions = [
  { value: 'osm', label: 'OSM' },
  { value: 'google', label: '谷歌' },
  { value: 'bing', label: '必应' },
  { value: 'ARCgis', label: 'ARC' },
  { value: 'tencent', label: '腾讯' },
  { value: 'tiandi', label: '天地图' },
  { value: 'mapbox', label: 'mapbox不让注册', disabled: true }
]
let model = null

onMounted(async () => {
  const viewer = new Cesium.Viewer("cesiumContainer", {
    infoBox: false,
    geocoder: false,
    homeButton: false,
    sceneModePicker: false, //是否显示投影方式控件
    baseLayerPicker: false,  //是否显示图层选择控件
    navigationHelpButton: false, //是否显示帮助信息控件
    geocoder: false, //是否显示地名查找控件
    animation: false, //是否显示动画控件
    timeline: false, //是否显示时间线控件
    fullscreenButton: false,
    vrButton: false,
    shadows: true
  })
  window.viewer = viewer
  viewer.cesiumWidget.creditContainer.style.display = "none"
})


async function changeMap(type) {
  
  let map = null
  if (type === 'gaode') {
    map = new Cesium.UrlTemplateImageryProvider({
      url: "http://webst02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8",
      minimumLevel: 4,
      maximumLevel: 18
    })

  } else if (type === 'osm') {
    map = new Cesium.OpenStreetMapImageryProvider({
      url: 'https://a.tile.openstreetmap.org/'
    })

  } else if (type === 'google') {
    map = new Cesium.UrlTemplateImageryProvider({
      url: "https://gac-geo.googlecnapps.cn/maps/vt?lyrs=s&x={x}&y={y}&z={z}"
    })

  } else if (type === 'bing') {
    map = await Cesium.BingMapsImageryProvider.fromUrl(
      "https://dev.virtualearth.net", {
      key: "AuKafWS8lQnTHXRdIBE7lyieYesaFPqSshgjjzyzPq9-Su7aQ_lu0MVEdoxvcQpO"
    })

  } else if (type === 'ARCgis') {
    map = await Cesium.ArcGisMapServerImageryProvider.fromUrl(
      'https://services.arcgisonline.com/arcgis/rest/services/World_Imagery/MapServer',
    )

  } else if (type === 'tencent') {
    map = new Cesium.UrlTemplateImageryProvider({
      url: "https://p2.map.gtimg.com/sateTiles/{z}/{sx}/{sy}/{x}_{reverseY}.jpg?version=400",
      customTags: {
        sx: function (imageryProvider, x, y, level) { return x >> 4; },
        sy: function (imageryProvider, x, y, level) { return ((1 << level) - y) >> 4 }
      }
    })

  } else if (type === 'tiandi') {
    map = new Cesium.UrlTemplateImageryProvider({
      url: 'http://{s}.tianditu.com/DataServer?T=img_w&X={x}&Y={y}&L={z}&tk=a197dc1151c64637e6bc687f7a4a92ca',
      subdomains: ['t0', 't1', 't2', 't3', 't4', 't5', 't6', 't7'],
      maximumLevel: 18,
      minimumLevel: 1,
      credit: 'Tianditu'
    })
  }

  if (map) {
    viewer.imageryLayers.addImageryProvider(map);
  }
}


function loadGLB() {
  if (!model) {
    const modelEntity = viewer.entities.add({
      name: "glb模型",
      position: new Cesium.Cartesian3.fromDegrees(106.585943, 29.562497),
      model: {
        uri: '/GuanZi_Test_02.glb',
        scale: 50,
        minimumPixelSize: 128,
        maxumunScale: 20000000,
      }
    })
    model = modelEntity
    model.userData = {}
    model.userData.position = [106.585943, 29.562497]
  }
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(model.userData.position[0], model.userData.position[1]-0.01, 1000),
    orientation: {
      pitch: Cesium.Math.toRadians(-50.0),
    }
  })
}
</script>

<style scoped>
#cesiumContainer {
  position: absolute;
  overflow: hidden;
  width: 100vw;
  height: 100vh;
  left: 0;
  top: 0;
}

.changeMap {
  position: fixed;
  top: 3%;
  left: 2%;
  width: 10%;
}

.loadModel {
  position: fixed;
  top: 7%;
  left: 2%;
  width: 10%;
}
</style>