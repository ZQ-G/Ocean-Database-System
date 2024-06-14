<template>
  <div id="map"></div>
</template>
  
<script setup>

import {ref,reactive,inject} from 'vue'
import {AddCircleOutline, RefreshCircleOutline, SearchOutline, TrashOutline, CreateOutline, DownloadOutline} from "@vicons/ionicons5"

import {useRouter, useRoute} from 'vue-router'

import { onMounted, onUnmounted } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";
const axios = inject("axios")

let map = null;
const paperList = ref([])

// const markersData = [
//   { name: "地点A", count: 10, positionStr: "(30.123, -60.456)" },
//   { name: "地点B", count: 20, positionStr: "(45.123, -75.456)" },
//   // 更多点的数据...
// ];

const markersData = ref([]);

function parseUsedNum(usedNum) {
  const coords = usedNum.match(/\((.*)\)/)[1].split(',').map(Number);
  return {
    latitude: coords[0], // 纬度
    longitude: coords[1] // 经度
  };
}
onMounted(async () => {
  const res = await axios.post(`/paper/search`);
  if (res.data.code == 200 && res.data.data.paper) {
    console.log("res.data.data.paper",res.data.data.paper);
    paperList.value = res.data.data.paper;
    // 将 paperList 转换为 markersData
    markersData.value = paperList.value.map(paper => ({
      name: paper.CreatureName, // 标记点名称
      count: paper.PaperNum,    // 标记点数量
      positionStr: paper.UsedNum // 经纬度坐标
    }));
    console.log("11111111111111",markersData.value);
  }

  window._AMapSecurityConfig = {
    securityJsCode: "d9cc468c03857152a1b3223e0a7c7055",
  };
  AMapLoader.load({
    key: "a69fae836455e236ff2dc639282f63ed", // 申请好的Web端开发者Key，首次调用 load 时必填
    version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
    plugins: ["AMap.Scale"], // 需要使用的插件列表，如比例尺'AMap.Scale'，支持添加多个如：['...','...']
  })
    .then((AMap) => {
      map = new AMap.Map("map", {
        // 设置地图容器id
        mapStyle: "amap://styles/whitesmoke", // 设置地图的显示样式
        // viewMode: "3D", // 是否为3D地图模式
        zoom: 1, // 初始化地图级别
        center: [180, 0], // 初始化地图中心点位置
      });

      const markerContent = `<div class="custom-content-marker">
                                <img src="//a.amap.com/jsapi_demos/static/demo-center/icons/dir-via-marker.png">
                              </div>`;
      const marker = new AMap.Marker({
        position: [180, 60],
        title: 'Hello World!',
        content: markerContent,
      });
      // 将标记添加到地图上
      // map.add(marker);

      createMarkers(AMap, markersData.value);
    })
    .catch((e) => {
      console.log(e);
    });
});

onUnmounted(() => {
  map?.destroy();
});


function createMarkers(AMap, data) {
  data.forEach(item => {
    // 去除空格并删除括号
    const positionStr = item.positionStr.trim().replace(/[()]/g, '');
    // 分割经纬度字符串并转换为数值
    const [latitude, longitude] = positionStr.split(",").map(Number);

    console.log("latitude",latitude);
    console.log("longitude",longitude);
    // 创建标记点
    const marker = new AMap.Marker({
      position: [longitude, latitude], // 高德地图的经纬度顺序是 [经度, 纬度]
      title: `${item.name} - 数量: ${item.count}`,
    });

    // 将标记添加到地图上
    map.add(marker);

    // 可以添加更多自定义属性或事件监听
  });
}
</script>

<style>
#map {
  width: 70vw;
  height: 80vh;
}
/* 这里可以添加一些样式，用于自定义地图或标记的样式 */
.custom-content-marker {
  width: 40px;
  height: 40px;
  background-color: #ffffff;
  border: 1px solid #000000;
  border-radius: 50%;
  text-align: center;
}
</style>
