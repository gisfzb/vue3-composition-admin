<template>
  <div
    id="map"
    ref="rootmap"
  />
</template>

<script lang="ts">
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
// import OSM from "ol/source/OSM";
import XYZ from 'ol/source/XYZ'
import Global from '../Global.vue'
// import {olmap} from '../Global'
// import {mapconfig} from  "../../config/mapconfig.js";
import { OverviewMap } from 'ol/control'
import {
  computed,
  defineComponent,
  ref,
  onBeforeMount,
  onBeforeUnmount,
  onMounted,
  reactive,
  toRefs
} from 'vue'
export default defineComponent({
  setup(props, context) {
    const obj = reactive({
      map: new Object()
    })
    let map
    function initMap() {
      const layer = new TileLayer({
        source: new XYZ({
          url: 'http://114.115.185.126:8016/mapway/tileserver/天地图影像/{z}/{x}/{y}'
        })
      })
      const map = new Map({
        target: 'map',
        layers: [layer],
        view: new View({
          projection: 'EPSG:4326', // 使用这个坐标系
          center: [104.04621661318255, 37.08339705294101], // 深圳
          zoom: 5
        })
      })

      const overview = new OverviewMap({
        collapsible: false
        // className:"OverviewMap_ctr"
      })
      // debugger
      map.addControl(overview)
      Global.olmap = map
      return map
    }
    onMounted(() => {
      map = initMap()
      // console.log("地图初始化")
    })
    return {
      map
    }
  }
})
</script>

<style>
#map {
  height: 100%;
}
/*隐藏ol的一些自带元素*/
.ol-attribution,
.ol-zoom {
  display: none;
}
</style>
