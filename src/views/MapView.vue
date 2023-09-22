<template>
  <div id="meteo-map"></div>
</template>

<script setup lang="ts">
import { onBeforeMount, onMounted, ref, type Ref } from 'vue'
import 'ol/ol.css'
import chroma from 'chroma-js'
import Map from 'ol/Map'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import XYZ from 'ol/source/XYZ'
import { Vector as VectorLayer } from 'ol/layer'
import { Vector as VectorSource } from 'ol/source'
import ImageLayer from 'ol/layer/Image'
import ImageCanvasSource from 'ol/source/ImageCanvas'
import { GeoJSON } from 'ol/format'
import Style from 'ol/style/Style'
import Stroke from 'ol/style/Stroke'
import { toLonLat } from 'ol/proj'

onMounted(() => {
  initMap()
})

// 矩阵数据 ROWS行数/COLUMNS列数
const ROWS = 232
const COLUMNS = 440

// 地图信息
const polarProjection = 'EPSG:4326'
const map: Ref<Map | null> = ref(null)
const vectorLayer: Ref<VectorLayer<VectorSource> | null> = ref(null)
const canvasLayer = ref<any>(null)
const imageArray = ref<Float32Array>()

const colors: any = chroma
  .scale([
    'rgb(149, 137, 211)',
    'rgb(149, 137, 211)',
    'rgb(149, 137, 211)',
    'rgb(149, 137, 211)',
    'rgb(150, 209, 216)',
    'rgb(129, 204, 197)',
    'rgb(103, 180, 186)',
    'rgb(95, 143, 197)',
    'rgb(80, 140, 62)',
    'rgb(121, 146, 28)',
    'rgb(171, 161, 14)',
    'rgb(223, 177, 6)',
    'rgb(243, 150, 6)',
    'rgb(236, 95, 21)',
    'rgb(190, 65, 18)',
    'rgb(138, 43, 10)',
    'rgb(138, 43, 10)'
  ])
  .domain([-20, -10, 0, 10, 20, 30, 40])

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
      zoom: 7,
      maxZoom: 14,
      minZoom: 7,
      enableRotation: false,
      center: [117.2346, 31.6267],
      extent: [111.5, 29.3, 123.8, 34.8]
    })
  })

  // 气象canvas图层
  canvasLayer.value = new ImageLayer({
    opacity: 0.7
  })
  map.value.addLayer(canvasLayer.value)

  initMeteo()

  const vectorSource = new VectorSource({
    format: new GeoJSON(),
    url: '/json/340000.json' // 替换为你的地理数据文件路径
  })

  vectorLayer.value = new VectorLayer({
    source: vectorSource
  })

  // 设置边界样式
  vectorLayer.value.setStyle(
    new Style({
      stroke: new Stroke({
        color: '#c9c9c9',
        width: 2
      })
    })
  )

  map.value.addLayer(vectorLayer.value)

  map.value.on('click', () => {})
}

const initMeteo = () => {
  fetch('/meteo/temperature.txt')
    .then((response) => {
      return response.text()
    })
    .then((res: any) => {
      const str = res.split(/\s+|\n+/)
      str.pop()
      const newData = str.map((item: any) => parseFloat(parseFloat(item).toFixed(2)))
      matrixFunction(newData)
    })
    .catch((error) => {
      console.log('Error:', error)
    })
}
const matrixFunction = (data: any) => {
  const img = new Image()
  img.crossOrigin = 'anonymous'
  const canvasPic = document.createElement('canvas')
  canvasPic.width = img.width
  canvasPic.height = img.height
  canvasPic.style.display = 'none'
  imageArray.value = new Float32Array(data.length)
  Float32Array.prototype.getValue = function (lon: any, lat: any) {
    // 经纬度范围
    const lonRange = [111.5, 123.8]
    const latRange = [29.3, 34.8]

    if (lon > 123.8 || lon < 111.5 || lat > 34.8 || lat < 29.3) {
      return null
    }
    // 将经纬度转换为栅格坐标
    const nx = ((lon - lonRange[0]) / (lonRange[1] - lonRange[0])) * (COLUMNS - 1)
    const ny = ((latRange[1] - lat) / (latRange[1] - latRange[0])) * (ROWS - 1)

    // 计算四点坐标
    let x0 = Math.floor(nx)
    let x1 = x0 + 1
    let y0 = Math.floor(ny)
    let y1 = y0 + 1

    // 防止超出范围
    x0 = Math.min(439, Math.max(0, x0))
    x1 = Math.min(439, Math.max(0, x1))
    y0 = Math.min(231, Math.max(0, y0))
    y1 = Math.min(231, Math.max(0, y1))

    // 计算权重
    const sx = nx - x0
    const sy = ny - y0

    // 获取四点值
    const v00 = this[x0 + y0 * COLUMNS]
    const v10 = this[x1 + y0 * COLUMNS]
    const v01 = this[x0 + y1 * COLUMNS]
    const v11 = this[x1 + y1 * COLUMNS]

    // 双线性插值
    return v00 * (1 - sx) * (1 - sy) + v10 * sx * (1 - sy) + v01 * (1 - sx) * sy + v11 * sx * sy
  }
  for (let i = 0; i < data.length; i++) {
    imageArray.value[i] = data[i]
  }

  canvasLayer.value.setSource(
    new ImageCanvasSource({
      canvasFunction,
      ratio: 1,
      projection: 'EPSG:4326'
    })
  )
}

const canvasFunction = (
  extent: any,
  resolution: any,
  pixelRatio: any,
  size: any,
  projection: any
) => {
  const width = Math.round(size[0]) * pixelRatio
  const height = Math.round(size[1]) * pixelRatio
  const can = document.createElement('canvas')
  can.width = width
  can.height = height
  const ctx = can.getContext('2d')
  const dx = Math.floor(6 * pixelRatio)
  const halfdx = Math.ceil(dx / 2)
  const newarr = []

  for (let j = 0; j <= height; j += dx) {
    for (let i = 0; i <= width; i += dx) {
      const coord = toLonLat(
        map.value!.getCoordinateFromPixel([i / pixelRatio, j / pixelRatio]),
        projection
      )
      const value = imageArray.value?.getValue(coord[0], coord[1])
      // const value = getMeteoValue(coord[0], coord[1])
      newarr.push({ lo: [coord[0], coord[1]], value })
      ctx!.fillStyle = colors(value).css()
      ctx!.fillRect(i - halfdx, j - halfdx, dx, dx)
    }
  }
  return can
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
