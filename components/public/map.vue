<template>
  <!-- <div id="container" class="container" :class="{fix: isfix}"></div> -->
  <div
    :id="id"
    :style="{width:width+'px',height:height+'px',margin:'32px auto'}"
    class="m-map"
    :class="{Fix : isFix, Abso: isAbso, abso:isabso}"
  />
</template>

<script>
import axios from "@/server/interface/utils/axios";
import { setTimeout } from "timers";
export default {
  props: {
    width: {
      type: Number,
      default: 300
    },
    height: {
      type: Number,
      default: 300
    },
    point: {
      type: Array,
      default() {
        return [115.494375, 40.06715];
      }
    },
    name: {
      type: String,
      default: ""
    },
    count: {
      type: Number,
      default: ""
    }
  },
  data() {
    return {
      id: `map`,
      isFix: false,  //固定定位
      isabso: false, //当到底部时
      isAbso: false, //在中间滚动时
      totalCount: "",
    };
  },
  watch: {
    name: function(newName, old) {},
    point: function(val, old) {
      if (this.map) {
        this.map.setCenter(val);
        this.marker.setPosition(val);
      }
    }
  },
  mounted() {
    window.Allcount = this.count;
    let self = this;
    self.id = `map${Math.random()
      .toString()
      .slice(4, 6)}`;
    window.onmaploaded = () => {
      let map = new window.AMap.Map(self.id, {
        resizeEnable: true,
        zoom: 11, //级别
        center: self.point //中心点坐标
      });
      self.map = map;
      //实时路况图层
      var trafficLayer = new window.AMap.TileLayer.Traffic({
        zIndex: 10
      });
      map.add(trafficLayer); //添加实时路况图层到地图

      // 定义插件
      window.AMap.plugin(["AMap.ToolBar", "AMap.OverView"], () => {
        let toolbar = new window.AMap.ToolBar({
          liteStyle: false //精简模式
        });
        map.addControl(toolbar);
        // map.addControl(
        //   new window.AMap.OverView({ isOpen: false})
        // );
        let marker = new window.AMap.Marker({
          icon: "https://webapi.amap.com/theme/v1.3/markers/n/mark_b.png",
          position: self.point
        });
        self.marker = marker;
        marker.setMap(map);

        // 鼠标点击marker弹出自定义的信息窗体
        AMap.event.addListener(marker, "click", function() {
          // infoWindow.open(map, marker.getPosition());
          openInfo();
        });

        // 实例化信息窗体
        function openInfo() {
          let info = [];
          info.push(self.name);
          let infoWindow = new window.AMap.InfoWindow({
            content: info.join("")
          });
          self.infoWindow = infoWindow;
          infoWindow.open(map, map.getCenter());
        }
      });
    };

    let url =
      "https://webapi.amap.com/maps?v=1.4.13&key=b8504b113da7ea9ad678bdc57c0e4a92&callback=onmaploaded";
    var jsapi = document.createElement("script");
    jsapi.charset = "utf-8";
    jsapi.src = url;
    document.head.appendChild(jsapi);

    window.addEventListener("scroll", function(e) {
      let h = document.documentElement.scrollTop || document.body.scrollTop;
      let moveH = (window.Allcount - 2) * 170.8 + 395 + 97;
      // 一开始还没移动到商品栏的时候
      if (h < 243) {
        self.isabso = false;
        self.isFix = false;
        self.isAbso = false;
        //移动到商品栏底部的时候
        // 变成绝对定位
      } else if (h > moveH) {
        self.isabso = true;
        self.isFix = false;
        self.isAbso = false;
        // 在头部和底部之间
      } else {
        self.isAbso = false;
        self.isFix = true;
        self.isabso = false;
      }
    });
  }
};
</script>

<style lang="scss">
// 美团的是一开始是static定位
// 然后开始移动到一定距离的时候变成fixed
// 最后超出距离就变成absolute定位
.Fix {
  position: fixed !important;
  right: 176px;
  top: -52px;
}
.Abso {
  position: absolute;
  left: 0px;
  top: 570px;
}
.abso{
  position: absolute !important;
  right: 0;
  bottom: -30px;
}
</style>


