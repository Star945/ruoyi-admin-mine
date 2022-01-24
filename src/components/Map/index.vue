<!--
 * @Author: your name
 * @Date: 2021-04-07 10:57:59
 * @LastEditTime: 2021-04-07 11:38:41
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \yunyi-yoga-admin\src\components\Map\index.vue
-->
<template>
  <div class="content">
    <div class="amap-wrapper">
      <el-amap-search-box
        class="search-box"
        :search-option="searchOption"
        :on-search-result="onSearchResult"
      ></el-amap-search-box>
      <el-amap
        class="amap-box"
        :zoom="zoom"
        :center="center"
        :events="events"
        :mapStyle="mapStyle"
      >
        <el-amap-marker
          v-for="(marker, i) in markers"
          :key="i"
          :position="marker"
        ></el-amap-marker>
      </el-amap>
    </div>
    <div class="Y-font-size-20 Y-margin-vertical-top-8">
      您选择的地址是：{{ address }}, 经纬度为：{{ lng }} , {{ lat }}
    </div>
    <!-- <div class="Y-text-align-right Y-avg-1">
      <el-button type="primary" @click="getLnglat">确 定</el-button>
    </div> -->
  </div>
</template>
<script>
import Vue from 'vue'
import VueAMap from 'vue-amap'
Vue.use(VueAMap)
//初始化
VueAMap.initAMapApiLoader({
  key: 'f712b4b0586afcb65ff0987b408b95e2',
  plugin: ['AMap.Autocomplete', 'AMap.PlaceSearch',
    'AMap.Scale', 'AMap.OverView', 'AMap.ToolBar', 'AMap.MapType', 'AMap.PolyEditor',
    'AMap.CircleEditor', 'Geocoder'],
  v: '1.4.4'
})
export default {
  data () {
    let _this = this
    return {
      center: [115.464804, 38.874062], //地图中心点坐标
      zoom: 12, //初始化地图显示层级
      mapStyle: "amap://styles/8b6be8ec497009e17a708205348b899a", //设置地图样式
      markers: [[115.464804, 38.874062]],
      searchOption: {
        city: "全国",
        citylimit: true,
      },
      address: "保定市",
      lng: 115.464804,
      lat: 38.874062,
      events: {
        moveend: () => { },
        zoomchange: () => { },
        click: (e) => {
          let { lng, lat } = e.lnglat
          this.center = [lng, lat]
          this.markers = [[lng, lat]]
          this.lng = lng
          this.lat = lat
          // 这里通过高德 SDK 完成。
          var geocoder = new AMap.Geocoder({
            radius: 1000,
            extensions: "all",
          })
          geocoder.getAddress([lng, lat], function (status, result) {
            if (status === "complete" && result.info === "OK") {
              if (result && result.regeocode) {
                _this.address = result.regeocode.formattedAddress
              }
            }
          })
        },
      },
      plugins: [
        {
          enableHighAccuracy: true, //是否使用高精度定位，默认:true
          timeout: 100, //超过10秒后停止定位，默认：无穷大
          maximumAge: 0, //定位结果缓存0毫秒，默认：0
          convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
          showButton: true, //显示定位按钮，默认：true
          buttonPosition: "LB", //定位按钮停靠位置，默认：'LB'，左下角
          showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
          showCircle: true, //定位成功后用圆圈表示定位精度范围，默认：true
          panToLocation: false, //定位成功后将定位到的位置作为地图中心点，默认：true
          zoomToAccuracy: true, //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：f
          extensions: "all",
          //地图定位按钮
          pName: "Geolocation",
          init (o) {
            // o 是高德地图定位插件实例
            o.getCurrentPosition((status, result) => {
              console.log(result)
              debugger
              if (result && result.position) {
                self.lng = result.position.lng
                self.lat = result.position.lat
                self.center = [self.lng, self.lat]
                self.loaded = true
                self.$nextTick()
              }
            })
          },
        },
      ]
    }
  },

  methods: {
    addMarker: function () {
      this.markers = []
      let lng = 121.5 + Math.round(Math.random() * 1000) / 10000
      let lat = 31.197646 + Math.round(Math.random() * 500) / 10000
      this.markers = [[lng, lat]]
    },
    onSearchResult (pois) {
      let latSum = 0
      let lngSum = 0
      if (pois.length > 0) {
        let center = pois[0]
        this.lng = center.lng
        this.lat = center.lat
        this.address = center.name
        this.center = [center.lng, center.lat]
        this.markers = [[center.lng, center.lat]]
      }
    },
    getLnglat () {

      this.$emit("getLnglat", this.lng, this.lat, this.address)
      return [this.lng, this.lat, this.address]
    },
  },
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.content {
  min-height: 400px;
  min-width: 400px;
}
.search-box {
  position: absolute;
  top: 25px;
  left: 20px;
}
.amap-wrapper {
  width: 100%;
  height: 500px;
  position: relative;
}
</style>