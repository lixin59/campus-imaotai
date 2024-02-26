<script lang="js">
import { onUnmounted, onMounted, shallowRef, reactive } from 'vue'
import AMapLoader from '@amap/amap-jsapi-loader'

export default {
  name: "Shop",
  props: {
    center: {
      type: Object,
      default: () => ({ lng: 106.674879, lat: 26.620664 })
    },
    positions: {
      type: Array,
      default: () => ([[106.674879, 26.620664]])
    },
    zoom: {
      type:Number,
      default: 15,
    },
    height: {
      type:String,
      default: '100%',
    },
    width: {
      type:String,
      default: '100%',
    },
  },
  emits: ['init'],
  setup(props){
    const map = shallowRef(null)
    const state = reactive({
      canRender: !!process.env.VUE_APP_GMAP_KEY
    })


    /** 初始化高德地图 */
    const initMap = () => {
      AMapLoader.load({
        key: process.env.VUE_APP_GMAP_KEY, // 申请好的Web端开发者Key，首次调用 load 时必填
        version: '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins: [] // 需要使用的的插件列表，如比例尺'AMap.Scale'等
      })
        .then((AMap) => {
          map.value = new AMap.Map('gmap-container', {
            //设置地图容器id
            // viewMode: '3D', //是否为3D地图模式
            resizeEnable: true,
            zoom: props.zoom, //初始化地图级别
            center: [props.center.lng, props.center.lat] //初始化地图中心点位置
          })
          AMap.plugin(['AMap.ToolBar', 'AMap.Scale', 'AMap.ControlBar'], function () {
            //异步同时加载多个插件
            const toolbar = new AMap.ToolBar()
            const scale = new AMap.Scale()
            const controlBar = new AMap.ControlBar({
              position: {
                top: '10px',
                right: '10px'
              }
            })
            map.value.addControl(toolbar)
            map.value.addControl(scale)
            map.value.addControl(controlBar)
          })
          addMarker(AMap)
        })
        .catch((e) => {
          console.log(e)
        })
    }
// 实例化点标记
    function addMarker(AMap) {
      const icon = new AMap.Icon({
        // 图标尺寸
        // size: new AMap.Size(50, 20),
        // 图标的取图地址
        image: '//a.amap.com/jsapi_demos/static/demo-center/icons/poi-marker-red.png',
        // 图标所用图片大小
        imageSize: new AMap.Size(30, 35)
        // 图标取图偏移量
        // imageOffset: new AMap.Pixel(-9, -3)
      })

      props.positions.forEach((p, i)=>{
      const  marker = new AMap.Marker({
          icon,
          position: new AMap.LngLat(p[0], p[1]),
          anchor: 'bottom-center',
          offset: new AMap.Pixel(0, 0)
        })
        marker.setMap(map)
        // // 设置鼠标划过点标记显示的文字提示
        // marker.setTitle('我是marker的title')
        // // 设置label标签
        // // label默认蓝框白底左上角显示，样式className为：amap-marker-label
        // marker.setLabel({
        //   direction: 'top',
        //   offset: new AMap.Pixel(10, 0), //设置文本标注偏移量
        //   content: "<div class='info'>我是 marker 的 label 标签</div>" //设置文本标注内容
        // })
        map.value.add(marker)

      })
    }
    onMounted(() => {
      initMap()
    })

    onUnmounted(() => {
      map.value?.destroy() // 销毁地图
    })

    return {
      state
    }
  }
}
</script>

<template>
  <div v-if="state.canRender" id="gmap-container" :style="{ height: height, width: width }"></div>
  <h1 v-else>请添加高德地图的key</h1>
</template>

<style scoped></style>

