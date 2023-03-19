<template>
  <div class="container">
    <!--为echarts准备一个dom容器-->
    <div ref="myEcharts" style="width: 1500px; height: 600px"></div>
  </div>
</template>
<script setup lang="ts">
import * as echarts from "echarts";
import worldMap from "@/assets/json/world.json";
import { ref, onMounted, onUnmounted, nextTick } from "vue";

// 世界地图画布
const myEcharts = ref();
const mapData = [
  {
    name: "CHN",
    projectTotal: 272,
    projectInvestment: 7382,
    projecUserCN: 23334,
  },
  {
    name: "USA",
    projectTotal: 72,
    projectInvestment: 382,
    projecUserCN: 3334,
  },
  {
    name: "RUS",
    projectTotal: 82,
    projectInvestment: 352,
    projecUserCN: 4334,
  },
];

const initMap = (mapData: any) => {
  nextTick(() => {
    echarts.registerMap("world", worldMap as any, {
      Alaska: {
        left: -131,
        top: 25,
        width: 15,
      },
      Hawaii: {
        left: -110,
        top: 28,
        width: 5,
      },
      "Puerto Rico": {
        left: -76,
        top: 26,
        width: 2,
      },
    });
    const myChart = echarts.init(myEcharts.value, "");
    const option = {
      title: {
        text: "境外事务大屏看板",
        left: "center",
      },
      tooltip: {
        triggerOn: "mousemove",
        formatter: function (params: any) {
          if (params?.data) {
            var a = `<span style="color:rgba(23, 240, 204)">国家名称：${params.data.name}</span></br>`;
            var b = `<span style="color:rgba(23, 240, 204)">项目总数量：${params.data.projectTotal}</span></br>`;
            var c = `<span style="color:rgba(23, 240, 204)">项目总投资额(亿元)：${params.data.projectInvestment}</span></br>`;
            var d = `<span style="color:rgba(23, 240, 204)">在册中方员工数：${params.data.projecUserCN}</span></br>`;
            return [a, b, c, d].join("");
          }
        },
        backgroundColor: "rgba(29, 38, 71)",
        // 额外附加的阴影
        extraCssText: "box-shadow:0 0 4px rgba(11, 19, 43,0.8);",
      },
      geo: {
        map: "world",
        // aspectScale: 0.75, //长宽比
        // zoom: 1.0,
        roam: false,
      },
      series: [
        {
          name: "境外事务",
          type: "map",
          roam: false,
          map: "world",
          // label: {
          // 	show: true,
          // },
          itemStyle: {
            normal: {
              borderColor: "rgb(147, 235, 248)",
              borderWidth: 1,
              areaColor: {
                type: "radial",
                x: 0.5,
                y: 0.5,
                r: 0.8,
                colorStops: [
                  {
                    offset: 0,
                    color: "#396a92", // 0% 处的颜色
                  },
                  {
                    offset: 1,
                    color: "#396a92", // 100% 处的颜色
                  },
                ],
                globalCoord: true, // 缺省为 false
              },
            },
            emphasis: {
              areaColor: "rgba(23, 240, 204)",
              shadowOffsetX: 0,
              shadowOffsetY: 0,
              borderWidth: 0,
            },
          },
          data: mapData,
        },
      ],
    };
    myChart.setOption(option);
    // 自动轮播
    let n = 0;
    let Carousel = setInterval(function () {
      myChart.dispatchAction({
        type: "showTip",
        seriesIndex: 0,
        name: mapData[n].name,
      });
      myChart.dispatchAction({
        type: "mapToggleSelect",
        seriesIndex: 0,
        name: mapData[n].name,
      });
      n++;
      if (n == mapData.length) {
        n = 0;
      }
    }, 3000);
    // 鼠标移入有mapData数据的国家清除定时器
    myChart.on("mouseover", function (params) {
      if (params.data) {
        clearInterval(Carousel);
      }
    });
    // 鼠标移出有mapData数据的国家清除定时器，并开起新的定时器
    myChart.on("mouseout", function (params) {
      if (params.data) {
        clearInterval(Carousel); //停止定时器
        Carousel = setInterval(function () {
          myChart.dispatchAction({
            type: "showTip",
            seriesIndex: 0,
            name: mapData[n].name,
          });
          myChart.dispatchAction({
            type: "mapToggleSelect",
            seriesIndex: 0,
            name: mapData[n].name,
          });
          n++;
          if (n == mapData.length) {
            n = 0;
          }
        }, 3000);
      }
    });
  });
};

//初始化echarts实例
onMounted(() => {
  initMap(mapData);
});
</script>
