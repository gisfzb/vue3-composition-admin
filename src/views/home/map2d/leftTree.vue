<template>
  <el-tree
    class="left_tree"
    :data="data"
    show-checkbox
    node-key="id"
    :default-expanded-keys="[1, 2, 3, 4]"
    :default-checked-keys="[]"
    :props="defaultProps"
    @check-change="checkChange"
  />
</template>

<script lang="ts">
import { Tile, Vector, Image } from 'ol/layer'
import { GeoJSON } from 'ol/format'
import { Style, Stroke } from 'ol/style'
import {
  TileWMS,
  ImageWMS,
  Vector as sourceVector,
  TileArcGISRest,
  ImageArcGISRest,
  XYZ
} from 'ol/source'
import Global from '@/components/Global.vue'
import { defineComponent } from 'vue'

export default defineComponent({
  setup() {
    const defaultProps = {
      children: 'children',
      label: 'label'
    }
    const data = [
      {
        id: 1,
        label: '基础数据',
        children: [
          {
            id: 4,
            label: '天地图',
            type: 'group',
            children: [
              {
                id: 9,
                type: 'tdt',
                label: '天地图影像'
              },
              {
                id: 10,
                type: 'tdt',
                label: '天地图矢量'
              },
              {
                id: 11,
                type: 'tdt',
                label: '天地图地形'
              }
            ]
          }
        ]
      },
      {
        id: 2,
        label: 'OGC服务',
        type: 'group',
        children: [
          {
            id: 5,
            label: 'WMS服务',
            type: 'wms',
            url: 'http://114.115.185.126:6081/geoserver/hainan/wms',
            layers: 'hainan:shi_list'
          },
          {
            id: 6,
            label: 'WFS服务',
            type: 'wfs',
            url: 'http://114.115.185.126:6081/geoserver/hainan/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=hainan%3Ashi_list&maxFeatures=50&outputFormat=application%2Fjson'
          },
          {
            id: '2-2',
            label: 'WMTS服务'
          }
        ]
      },
      {
        id: 3,
        label: 'Arcgis服务',
        type: 'group',
        children: [
          {
            id: 7,
            type: 'arcgis',
            label: '动态服务',
            url: 'http://www.srhgis.cn:6080/arcgis/rest/services/地学大数据/ChinaBasin/MapServer',
            layers: null
          },
          {
            id: 8,
            type: 'arcgis',
            label: '要素服务'
          },
          {
            id: 8,
            type: 'arcgisTile',
            label: '切片服务',
            url: 'https://hhly.xblcp.cn:7033/mapway/tileserver/arcgis/shidi26'
          }
        ]
      }
    ]

    function arrayToTree(items: any): Array<Object> {
      const result = [] // 存放结果集
      const itemMap = {} //
      for (const item of items) {
        const id = item.id
        const pid = item.pid

        if (!itemMap[id]) {
          itemMap[id] = {
            children: []
          }
        }

        itemMap[id] = {
          ...item,
          children: itemMap[id].children
        }

        const treeItem = itemMap[id]

        if (pid === 0) {
          result.push(treeItem)
        } else {
          if (!itemMap[pid]) {
            itemMap[pid] = {
              children: []
            }
          }
          itemMap[pid].children.push(treeItem)
        }
      }
      return result
    }

    function checkChange(data: any, checked: boolean, chilChecked: boolean) {
      console.log(data, checked, chilChecked)
      if (checked && data.type != 'group') {
        if (data.type === 'wms') {
          const sjLayer = new Image({
            className: '',
            source: new ImageWMS({
              url: data.url,
              params: {
                FORMAT: 'image/png',
                LAYERS: data.layers
              }
            })
          })
          data.layer = sjLayer
          Global.olmap.addLayer(sjLayer)
        } else if (data.type === 'wfs') {
          console.log('加载wfs服务')
          const vector = new Vector({
            // 数据来源
            source: new sourceVector({
              format: new GeoJSON(),
              url: data.url
            }),
            // layer样式
            style: function(feature, resolution) {
              return new Style({
                stroke: new Stroke({
                  color: 'blue',
                  width: 1
                })
              })
            }
          })
          data.layer = vector
          Global.olmap.addLayer(vector)
        } else if (data.type === 'arcgis') {
          console.log('加载arcgis服务')
          const layer = new Tile({
            source: new TileArcGISRest({
              url: data.url
            })
          })
          data.layer = layer
          Global.olmap.addLayer(layer)
        } else if (data.type === 'arcgisTile') {
          console.log('加载arcgis切片服务')
          const layer = new Tile({
            source: new XYZ({
              url: data.url + '/tile/{z}/{y}/{x}'
            })
          })
          data.layer = layer
          Global.olmap.addLayer(layer)
        }
      } else {
        // 取消显示
        if (data.layer) {
          data.layer.setVisible(false)
        }
      }
    }
    return {
      data,
      checkChange
    }
  }
})
</script>

<style scoped>
.left_tree {
  height: 80%;
  width: 25%;
  min-width: 180px;
  position: absolute;
  background: #f5f7fa;
  z-index: 99999;
  top: 20px;
  left: 20px;
  overflow-y: auto;
}
</style>
