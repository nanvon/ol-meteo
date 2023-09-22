<template>
  <div id="meteo-map"></div>
</template>

<script setup lang="ts">
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import XYZ from 'ol/source/XYZ'
import { onBeforeMount, onMounted, ref, type Ref } from 'vue'

onMounted(() => {
  initMap()
})

const polarProjection = 'EPSG:4326'
const map: Ref<Map | null> = ref(null)

const initMap = () => {
  map.value = new Map({
    target: 'meteo-map',
    layers: [
      new TileLayer({
        source: new XYZ({
          url: 'http://webst0{1-4}.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}'
        })
      })
    ],
    view: new View({
      projection: polarProjection,
      zoom: 8,
      maxZoom: 14,
      minZoom: 7,
      enableRotation: false,
      center: [117.2346, 31.6267],
      extent: [111.5, 29.3, 123.8, 34.8]
    })
  })
}

onBeforeMount(() => {
  if (map.value) {
    // 解除地图与HTML元素的关联
    map.value.setTarget(null as any)

    // 销毁地图及其相关资源
    map.value.dispose()

    // 将地图变量设为null，释放内存
    map.value = null
  }
})
</script>

<style lang="less" scoped>
#meteo-map {
  height: calc(100vh);
  width: auto;
}
</style>
