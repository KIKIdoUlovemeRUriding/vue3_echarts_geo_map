<template>
  <div
    id="main"
    class="main"
  >
    <div class="btn-list">
      <button @click="switchData(scatterSeries)">散点图</button>
      <button @click="switchData(heatmapSeries)">热力图</button>
      <button @click="switchData(lineseries)">路径图</button>
      <button @click="handleMockBarData">柱形图</button>
      <button @click="handleChangeMap">
        {{ currentJson === ShandongJson ? '切换到青岛市' : '切换到山东省' }}
      </button>
    </div>
    <div id="map-container">
      <GeoMap
        ref="map"
        :series="currentData"
        :height="600"
        :json="currentJson"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, ref } from 'vue';
import * as echarts from 'echarts';
// 引入地图JSON用来切换地区
import ShandongJson from './assets/json/shandong.json?raw';
import QindaoJson from './assets/json/qingdao.json?raw';
import GeoMap from './components/GeoMap.vue';

// map子组件的模板引用，我们通过这里可以访问到子组件的echarts实例
const map = ref<any>(null);

// 演示不同数据类型的地图
const currentData = ref();
// 演示不同地区的地图
const currentJson = ref(ShandongJson);
const switchData = (data: any) => {
  currentData.value = data;
};

/**
 * 地图中心点, 通常地图json中有确定的中心点，实际应用中我们需要根据具体视图UI来自定义中心点
 * json数据里的center: [*****, *****]就是中心点的经纬度数组，可以取来改动和使用
 * 中心点主要在用在，给不含有经纬度的数据集，通过遍历赋值，添加上我们预设的中心点经纬度
 */
const regionCenterPoints = [
  {
    name: '青岛市',
    value: [120.355173, 36.082982],
  },
  {
    name: '烟台市',
    value: [121.391382, 37.539297],
  },
  {
    name: '济南市',
    value: [117.000923, 36.675807],
  },
];
console.log(regionCenterPoints);
// 父组件把处理好的series传给子组件, 这里展示几种业务常用的图表类型

/**
 * 1. 散点图
 * 数据格式： [{name: '数据名称', value： [经度, 纬度]}]，name是可选项
 * 默认散点大小是一样大，即symbolSize为10，自行设置symbolSize可以控制点的大小
 * 散点的颜色，如果map子组件里声明了visualMap，将会使用visualMap inrange的第一个颜色，
 * 如果要更改，需要在data数据里传入itemStyle更改颜色
 * 其他散点图的参数参考 https://echarts.apache.org/zh/option.html#series-scatter
 */
const scatterSeries = ref<echarts.ScatterSeriesOption>({
  type: 'scatter',
  coordinateSystem: 'geo',
  z: 10,
  data: [
    {
      name: '青岛市',
      value: [120.355173, 36.082982],
      symbolSize: 30,
      itemStyle: {
        color: 'red',
      },
    },
    {
      name: '烟台市',
      value: [121.391382, 37.539297],
      symbolSize: 80,
    },
    {
      name: '济南市',
      value: [117.000923, 36.675807],
    },
  ],
  tooltip: {
    show: true,
    // echarts这个formatter函数参数的类型声明很多没有导出，直接用any
    formatter: (params: any) => {
      return `<div style="background: red">${params?.data.value[0]}</div>`;
    },
  },
});

/**
 * 2. 热力图
 * 数据格式：[[经度,纬度,热力值],...]
 * 使用热力图必须要在map子组件里定义visualMap，否则无法显示
 * 其他热力图的参数参考 https://echarts.apache.org/zh/option.html#series-heatmap
 *
 */
const heatmapSeries = ref<echarts.HeatmapSeriesOption>({
  type: 'heatmap',
  coordinateSystem: 'geo',
  data: [
    [120.355173, 36.082982, 100],
    [121.391382, 37.539297, 100],
    [119.121733, 36.146036, 100],
    [119.121733, 36.146037, 100],
    [119.121733, 36.146038, 100],
    [119.121733, 36.146039, 100],
    [119.121733, 36.146031, 100],
    [119.121733, 36.146032, 100],
    [119.121733, 36.146033, 100],
  ],
  z: 8,
  geoIndex: 0,
  blurSize: 20,
});

/**
 * 3. 路径图
 * 数据格式：[{coords: [[经度,纬度],[经度,纬度]], lineStyle: {}}] , coord是二维数组
 * coords数组第一项为起始点的经纬度，第二项为终点的经纬度，lineStyle是线的样式可选项
 * 其他路径图的参数参考 https://echarts.apache.org/zh/option.html#series-lines
 */
const lineseries = ref<echarts.LinesSeriesOption>({
  type: 'lines',
  coordinateSystem: 'geo',
  data: [
    {
      coords: [
        [120.355173, 36.082982],
        [121.391382, 37.539297],
      ],
      lineStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          {
            offset: 0,
            color: '#3B80E2',
          },
          {
            offset: 1,
            color: '#49BEE5',
          },
        ]),
        width: 2,
      },
    },
    {
      coords: [
        [120.355173, 36.082982],
        [117.000923, 36.675807],
      ],
      lineStyle: {
        color: 'darkgreen',
        width: 2,
      },
    },
  ],
  lineStyle: {
    curveness: 0.3, // 线的曲度，默认为0直线，0-1的取值范围，值越大曲度越大，可以为负数即设置为反向弧度
  },
  effect: {
    show: true, // 是否显示特效
    period: 6, // 特效动画时间，单位s
    trailLength: 0.9, // 特效尾迹长度，取值0-1，值越大尾迹越长
    constantSpeed: 40, // 特效动画速度，值为像素点，越大越快
    color: 'lightblue', // 特效标记的颜色
    symbol: 'circle', // 特效标记的图形，可以使用图片等
    symbolSize: 10, // 特效标记的大小
  },
});
/**
 * 4. 柱形图
 * 柱形图稍微复杂一些，由于coordinateSystem无法设置geo,只能设置为直角坐标系
 * 所以在这里要做一下geo经纬度到二维坐标的转换才能使用
 * 处理好的数据并不是series数据，而是需要重新执行setOption
 * 其他柱形图的参数参考 https://echarts.apache.org/zh/option.html#series-bar
 * 通过该示例，其余大部分的二维坐标的图表都可以自行摸索实现
 */

const mockBarData = [
  {
    name: '青岛市',
    value: [120.355173, 36.082982],
    count: 100,
  },
  {
    name: '烟台市',
    value: [121.391382, 37.539297],
    count: 10,
  },
  {
    name: '济南市',
    value: [117.000923, 36.675807],
    count: 20,
  },
];

const barSeries = ref<any>({
  xAxis: [],
  yAxis: [],
  series: [],
  grid: [],
});
const handleMockBarData = () => {
  mockBarData.forEach((item: any, index: number) => {
    const geoCoords = item.value; // 获取经纬度
    // 转换后的二维坐标
    const coords: any = map.value?.chartInstance?.convertToPixel(
      'geo',
      geoCoords
    );
    index += 1;
    barSeries.value.xAxis.push({
      id: index, // 组件id，在配置中引用标识
      gridIndex: index, // x轴所在的grid的索引
      gridId: index,
      type: 'category', // 设置为主轴
      splitArea: {
        show: false,
      },
      splitLine: {
        // 是否显示坐标轴在 grid 区域中的分隔线
        show: false,
      },
      axisTick: {
        // 是否显示坐标轴刻度
        show: false,
      },
      axisLabel: {
        // 是否显示坐标轴刻度标签
        show: false,
      },

      axisLine: {
        // 是否显示坐标轴轴线
        show: false,
      },
      z: 100,
      // 通常设置一个比较大的数值 确保柱形图在最高层
    });
    // Y轴配置
    barSeries.value.yAxis.push({
      id: index, // 组件id，在配置中引用标识
      gridIndex: index, // x轴所在的grid的索引
      gridId: index,
      // 默认柱子的高度是相同的，如果要设置为不同，调整min 和 max
      // min: 0.1,
      // max: 60,
      // interval: 0.1,
      splitLine: {
        // 坐标轴在 grid 区域中的分隔线
        show: false,
      },
      axisTick: {
        // 坐标轴刻度
        show: false,
      },
      axisLabel: {
        // 坐标轴刻度标签
        show: false,
      },
      axisLine: {
        // 坐标轴轴线
        show: false,
      },
      splitArea: {
        show: false,
      },
      z: 100,
    });
    // 坐标系配置控制柱子的位置，需要根据实际UI视图来调整 left top
    barSeries.value.grid.push({
      gridIndex: index, // x轴所在的grid的索引
      gridId: index,
      id: index, // 组件id，在配置中引用标识
      width: 100, // 组件的宽度
      height: 100, // 组件的高度
      left: coords[0] - 50, // 离容器左侧的距离
      top: coords[1] - 105, // 离容器上侧的距离
      z: 100,
    });
    barSeries.value.series.push({
      labelLine: {
        show: false,
      },
      emphasis: {
        show: false,
        disabled: true,
      },
      z: 100,
      id: index, // 组件id，在配置中引用标识
      type: 'bar', // 柱状图
      xAxisId: index, // 使用的x轴的id
      yAxisId: index, // 使用的y轴的id
      barGap: 0, // 柱间距离
      barWidth: 8,
      barCategoryGap: 0, // 同一系列的柱间距离
      data: [item.count], // 柱子数据, 可以设置多个数据，显示多个柱子
      label: {
        show: false,
        itemStyle: {
          color: 'red',
        },
      },
      itemStyle: {
        // 柱子样式
        borderRadius: [4, 4, 0, 0],
        color: new echarts.graphic.LinearGradient(
          0,
          0,
          0,
          1,
          [
            {
              offset: 0,
              color: 'rgba(232, 137, 58, 1)',
            },
            {
              offset: 1,
              color: 'rgba(232, 137, 58, 0.2)',
            },
          ],
          false
        ),
      },
    });
  });
  // 执行setOption, 通常会和之前的图表合并，如果不想合并，传递第二个参数{replaceMerge: 'series'}
  map.value.chartInstance.setOption(barSeries.value, {
    replaceMerge: 'series',
  });
};

/**
 * 关于缩放和拖动的问题，echarts自带的roam问题很多：
 * 问题1：地图贴图之后，只有地图本身，表面没有图表的时候，只有在地图空白区域才可以拖动和缩放
 * 问题2：地图表面加入图表后，拖动和缩放只对第一层geo生效
 * 问题3：通过使用监听事件georoam，对除了第二层geo的地图进行拖动和缩放，会卡顿无比（GeoMap.vue）
 * 问题4：缩放和拖动对geo经纬转成二维坐标的图表不生效，例如上面的柱形图
 * 综上自己实现一个wheel监听,缩放父级dom元素，算是一个折中的解决方案
 */
let num = 1;
const handleMouseEvent = () => {
  const main = document.getElementById('main');
  const mapContainer = document.getElementById('map-container');

  main?.addEventListener('wheel', (event: WheelEvent) => {
    // 该缩放方案tooltip会出现移动位置的问题，所以直接把显示中的tooltip隐藏了
    map.value?.chartInstance?.dispatchAction({
      type: 'hideTip',
    });
    event.preventDefault();
    if (event.deltaY < 0) {
      num += 0.1;
      mapContainer!.style.transform = `scale(${num})`;
    } else {
      if (num < 0.11) {
        num = 0.1;
      } else {
        num -= 0.1;
      }
      mapContainer!.style.transform = `scale(${num})`;
    }
  });
};

// 切换地图JSON即可完成切换
const handleChangeMap = () => {
  currentJson.value =
    currentJson.value === ShandongJson ? QindaoJson : ShandongJson;
};
// 监听地图的点击事件
const handleClickMap = () => {
  map.value?.chartInstance?.on('click', (params: any) => {
    if (params.componentType === 'geo') {
      if (params.name === '青岛市') {
        currentJson.value = QindaoJson;
      }
    }
  });
};
// 监听地图的选中事件
const handleSelectMap = () => {
  map.value?.chartInstance?.on('geoselectchanged', (params: any) => {
    console.log(params.selected, 'selectchanged');
  });
};

onMounted(() => {
  handleMouseEvent();
  handleClickMap();
  handleSelectMap();
});
</script>

<style lang="less">
.tooltip-style {
  width: 100px;
  height: 33px;
  padding: 5px;
  border-radius: 8px;
  background: darkorange;
  color: #fff;
  line-height: 33px;
  p {
    margin: 0;
  }
}
.main {
  width: 1000px;
  height: 1000px;
  position: relative;
  .btn-list {
    width: 500px;
    display: flex;
    position: relative;
    z-index: 1000;
    justify-content: space-between;
  }
  #map-container {
    position: absolute;
    transition: all 0.1s linear;
  }
}
</style>
