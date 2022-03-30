<template>
  <div class="tools">
    <el-button-group>
      <el-button
        type="primary"
        @click="drawLineString"
      >
        测距
      </el-button>
      <el-button
        type="primary"
        @click="drawPolygon"
      >
        测面
      </el-button>
      <el-button
        type="primary"
        @click="disableDraw"
      >
        清除
      </el-button>
    </el-button-group>
  </div>
</template>

<script lang='ts'>
import {
  computed,
  defineComponent,
  onBeforeMount,
  onBeforeUnmount,
  onMounted,
  reactive,
  ref,
  toRefs
} from 'vue'
import { Vector as VectorSource } from 'ol/source'
import { Vector as VectorLayer } from 'ol/layer'
import { Style, Fill, Stroke, Circle } from 'ol/style'
import { Polygon, LineString } from 'ol/geom'
import { getArea, getLength } from 'ol/sphere'
import { Draw } from 'ol/interaction'
import { Map, Overlay, Feature } from 'ol'
import { unByKey } from 'ol/Observable'
import Global from '@/components/Global.vue'
export default defineComponent({
  setup(props, context) {
    let map: Map
    let helpTooltipElement: any
    let featureType = 'LineString'
    let measureTooltipElement: any
    let measureTooltip: any
    let draw: Draw, listener: any
    // let source: VectorSource;
    let helpTooltip: Overlay
    // let drawingFeature: any;
    // let pointerMoveHandler: any;

    // 绘制结果矢量图层
    const source = new VectorSource()
    const vectorLayer = new VectorLayer({
      source: source,
      // 矢量图层样式
      style: new Style({
        fill: new Fill({
          color: 'rgba(125, 125, 125, 0.2)'
        }),
        stroke: new Stroke({
          color: '#ffcc33',
          width: 2
        }),
        image: new Circle({
          radius: 7,
          fill: new Fill({
            color: '#ffcc33'
          })
        })
      })
    })

    // 鼠标移动事件
    let drawingFeature: any = null
    let helpMsg = '点击继续绘制'
    const lineHelpMsg = '点击继续绘制线'
    const polygonHelpMsg = '点击继续绘制面'
    const pointerMoveHandler = (evt: any) => {
      if (evt.dragging) {
        return
      }
      if (drawingFeature) {
        const geom = drawingFeature.getGeometry()
        if (geom instanceof Polygon) {
          helpMsg = polygonHelpMsg
          console.log('polygon')
        } else if (geom instanceof LineString) {
          helpMsg = lineHelpMsg
          console.log('linestring')
        }
      }
      helpTooltipElement.innerHTML = helpMsg
      helpTooltip.setPosition(evt.coordinate)
      helpTooltipElement.classList.remove('hidden')
    }

    // map.on('pointermove', pointerMoveHandler)
    // map.getViewport().addEventListener('mouseout', () => {
    //     helpTooltipElement.classList.add('hidden')
    // })

    // 对绘制线或面按钮添加点击事件
    // document.getElementById('measureDistance').addEventListener('click', () => {
    //     featureType = 'LineString'
    //     map.removeInteraction(draw)
    //     addInteraction()
    // })
    // document.getElementById('measureArea').addEventListener('click', () => {
    //     featureType = 'Polygon'
    //     map.removeInteraction(draw)
    //     addInteraction()
    // })

    // 格式化输出线的长度和多边形的面积
    const formatLength = (line: any) => {
      const length = getLength(line)
      let output
      if (length > 100) {
        output = Math.round((length / 1000) * 100) / 100 + ' km'
      } else {
        output = Math.round(length * 100) / 100 + ' m'
      }
      return output
    }
    const formatArea = (polygon: any) => {
      const area = getArea(polygon)
      let output
      if (area > 10000) {
        output = Math.round((area / 1000000) * 100) / 100 + ' km<sup>2</sup>'
      } else {
        output = Math.round(area * 100) / 100 + ' m<sup>2</sup>'
      }
      return output
    }

    // 添加绘制控件
    function addInteraction() {
      const type = featureType
      draw = new Draw({
        source: source,
        type: type,
        // 勾绘出要素的样式
        style: new Style({
          fill: new Fill({
            color: 'rgba(125, 125, 125, 0.2)'
          }),
          stroke: new Stroke({
            color: 'rgba(0, 0, 0, 0.5)',
            lineDash: [10, 10],
            width: 2
          }),
          image: new Circle({
            radius: 5,
            stroke: new Stroke({
              color: 'rgba(0, 0, 0, 0.7)'
            }),
            fill: new Fill({
              color: 'rgba(255, 255, 255, 0.2)'
            })
          })
        })
      })
      map.addInteraction(draw)

      try {
        createMeasureTooltip()
        createHelpTooltip()
      } catch (error) {
        console.log('error', error)
      }

      draw.on(
        'drawstart',
        (evt: any) => {
          drawingFeature = evt.feature
          let tooltipCoord = evt.coordinate
          listener = drawingFeature.getGeometry().on('change', (evt: any) => {
            const geom = evt.target
            let output
            if (geom instanceof Polygon) {
              output = formatArea(geom)
              tooltipCoord = geom.getInteriorPoint().getCoordinates()
            } else if (geom instanceof LineString) {
              output = formatLength(geom)
              tooltipCoord = geom.getLastCoordinate()
            }
            measureTooltipElement.innerHTML = output
            measureTooltip.setPosition(tooltipCoord)
          })
        }
        // this
      )
      draw.on(
        'drawend',
        (evt: any) => {
          measureTooltipElement.className = 'tooltip tooltip-static'
          measureTooltip.setOffset([0, -7])
          drawingFeature = null
          measureTooltipElement = null
          // createMeasureTooltip()  //开启自动测量
          map.removeInteraction(draw) // 注销地图事件
          // if(helpTooltipElement){
          // 	helpTooltipElement.classList.add('hidden')
          // }
          map.removeOverlay(helpTooltip)
          // map.off('pointermove', pointerMoveHandler)
          map.getViewport().removeEventListener('mouseout', () => {
            helpTooltipElement.classList.add('hidden')
          })
          unByKey(listener)
        }
        // this
      )

      map.on('pointermove', pointerMoveHandler)
      map.getViewport().addEventListener('mouseout', () => {
        helpTooltipElement.classList.add('hidden')
      })
    }

    // 创建帮助信息标签
    function createHelpTooltip() {
      if (helpTooltipElement) {
        helpTooltipElement.parentNode.removeChild(helpTooltipElement)
      }
      helpTooltipElement = document.createElement('div')
      helpTooltipElement.className = 'tooltip hidden'
      helpTooltip = new Overlay({
        element: helpTooltipElement,
        offset: [15, 0],
        positioning: 'center-left'
      })
      map.addOverlay(helpTooltip)
    }

    // 创建测量结果信息标签
    function createMeasureTooltip() {
      if (measureTooltipElement) {
        measureTooltipElement.parentNode.removeChild(measureTooltipElement)
      }
      measureTooltipElement = document.createElement('div')
      measureTooltipElement.className = 'tooltip tooltip-measure'
      measureTooltip = new Overlay({
        element: measureTooltipElement,
        offset: [0, -15],
        positioning: 'bottom-center'
      })
      map.addOverlay(measureTooltip)
    }

    const drawLineString = () => {
      featureType = 'LineString'
      map.removeInteraction(draw)
      addInteraction()
    }

    const drawPolygon = () => {
      featureType = 'Polygon'
      map.removeInteraction(draw)
      addInteraction()
    }

    const disableDraw = () => {
      source.refresh() // 刷新源，源将被清除

      //  measureTooltipElement.className = "tooltip tooltip-static";
      measureTooltip.setOffset([0, -7])
      drawingFeature = null
      measureTooltipElement = null
      // createMeasureTooltip()  //开启自动测量
      map.removeInteraction(draw) // 注销地图事件
      // if(helpTooltipElement){
            	helpTooltipElement.classList.add('hidden')
      // }
      map.removeOverlay(helpTooltip)
      map.removeOverlay(measureTooltip)
      // map.off('pointermove', pointerMoveHandler)
      map.getViewport().removeEventListener('mouseout', () => {
        helpTooltipElement.classList.add('hidden')
      })
      unByKey(listener)
    }

    // 初始化方法
    onMounted(() => {
      map = Global.olmap
      map.addLayer(vectorLayer)
      // initEvent(); // 初始化
      createHelpTooltip()
      createMeasureTooltip()
      // addInteraction()
    })

    return {
      // addInteraction,
      drawLineString,
      drawPolygon,
      disableDraw
    }
  }
})
</script>

<style lang="scss" scoped>
.tools {
  position: absolute;
  background: #f5f7fa;
  z-index: 99999;
  top: 20px;
  right: 20px;
}
</style>>
