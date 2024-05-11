<template>
  <div
    :style="{ height: `${props.height}px` }"
    class="map-container"
    ref="map"
  ></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, withDefaults, watch } from 'vue';
import * as echarts from 'echarts';
/**
 * 引入地图JSON数据
 * 移步阿里dataV获取其他地图数据 ↓
 * https://datav.aliyun.com/portal/school/atlas/area_selector
 *
 * 以raw后缀声明引入的是JSON字符串，在echarts.registerMap使用JSON.parse解析
 * 这么操作的目的是解决直接import json并在echarts.registerMap中使用，产生的echarts ts类型报错，
 * 其实不用raw后缀引入JSON，并在echarts.registerMap里直接使用JSON对象，功能上不会有任何问题，
 * 只需要
 * import ShandongGeoJson from '../assets/json/shandong.json';
 * echarts.registerMap('shandong', ShandongGeoJson as any)即可
 */
import ShandongGeoJson from '../assets/json/shandong.json?raw';

// 父级传值来配置地图
interface Props {
  // 地图的大小主要依赖DOM容器的高度
  height?: number;
  json: string;
  // 具体业务数据（散点图 热力图 柱形图等）
  series: any;
}
const props = withDefaults(defineProps<Props>(), {
  height: 400,
  json: ShandongGeoJson,
  series: null,
});

// echats挂载的DOM节点
const map = ref<HTMLElement>();

// echarts实例
const chartInstance = ref<echarts.ECharts>();
// 抛出echarts实例，方便父组件使用
defineExpose({
  chartInstance,
});
// 注册地图
const registerMap = (value?: echarts.SeriesOption) => {
  echarts.registerMap('shandong', JSON.parse(props.json));
  if (chartInstance.value) {
    // 判断options是否为空，为空则执行初始渲染，不为空则执行更新
    const options = chartInstance.value.getOption();
    if (options) {
      options.series = value;
      chartInstance.value.setOption(options);
    } else {
      drawSticker(value);
    }
  } else {
    console.error('echarts实例未初始化，无法渲染地图');
  }
};

/**
 * 绘制地图
 * 绘制地图贴图要早于渲染地图，否则地图贴图会出现黑色贴图的情况
 *
 */
const drawSticker = (value?: echarts.SeriesOption) => {
  let sticker: HTMLCanvasElement;
  const width = map.value?.offsetWidth ?? 1000;
  const stickerDom = document.getElementById('sticker') as HTMLCanvasElement;
  if (!stickerDom) {
    sticker = document.createElement('canvas');
    sticker.id = 'sticker';
    map.value?.appendChild(sticker);
  } else {
    sticker = stickerDom;
  }
  // 贴图的大小，宽高一致，一般情况下小于props.height更为合适，根据实际情况调整
  sticker.width = props.height;
  sticker.height = props.height;
  const stickerCtx = sticker.getContext('2d');
  const image = new Image();
  image.src = new URL('../assets/image/sticker.jpg', import.meta.url).href;
  image.onload = () => {
    // 5个参数分别为：图片、x原点、y原点、宽度、高度
    stickerCtx?.drawImage(image, 0, 0, width, width);
    // 不显示贴图在文档流中
    sticker.style.display = 'none';
    generateMap(value);
  };
};

const generateMap = (value?: echarts.SeriesOption) => {
  const echartsOptions = {
    // 一般背景一定设置为透明
    backgroundColor: 'transparent',
    /**
     * 这里的tooltip为《地图本身》鼠标移入显示的div,这里设置为false，不会影响散点图 热力图等series设置的tooltip
     */
    tooltip: {
      show: true,
    },
    grid: {
      top: 0,
      bottom: 0,
      left: 0,
      right: 0,
    },
    geo: [
      {
        map: 'shandong',
        roam: false,
        selectedMode: 'multiple',
        left: '18%',
        top: '18%',
        // 应用在第一层的tooltip
        tooltip: {
          show: true,
          // 使用自定义的tooltip样式，下面这四个属性要这么设置，避免影响到自己的样式
          backgroundColor: 'transparent',
          borderWidth: 0,
          padding: 0,
          extraCssText: 'box-shadow: none',
          // className: 'tooltip-style',
          // 自定义的tooltip样式，可以在《公共css文件》声明css类在此使用，当然也可以直接使用style行内样式
          formatter: (params: any) => {
            return `
            <div class="tooltip-style">
              <p>
                ${params?.name}  
              </p>
            </div>
        `;
          },
        },
        // 可以给某个地区单独设置样式
        regions: [
          {
            name: '青岛市', // 必须提供对应的name
            itemStyle: {},
          },
        ],
        label: {
          show: true,
          fontSize: 12,
          // fontFamily: 'YSXS',
        },
        itemStyle: {
          borderColor: 'red',
          borderWidth: 1,
          color: {
            image: document.getElementById('sticker'),
            repeat: 'no-repeat',
          },
        },
        z: 2,
      },
      {
        map: 'shandong', //注册地图的名字
        roam: false, //开启鼠标缩放和平移漫游。默认不开启
        left: '18.8%',
        top: '18.8%',
        label: {
          show: false
        },
        itemStyle: {
          shadowColor: '#000',
          shadowOffsetX: 6,
          shadowOffsetY: 6,
          shadowBlur: 3,
          color: 'darkgreen', // 背景
          borderWidth: 1, // 边框宽度
          borderColor: 'red', // 边框颜色
          fontSize: 0.1, //
        },
        z: 1,
      },
    ],
    visualMap: {
      type: 'continuous',
      map: 'shandong',
      show: false,
      top: 'top',
      min: 1,
      max: 1 * 100,
      // seriesIndex: 0,
      splitNumber: 5,
      inRange: {
        color: ['#d94e5d', '#eac736', '#50a3ba'].reverse(),
      },
      calculable: true,
    },
    series: [value],
    /**
     *  需要设置最外层和内部的边框不同颜色时，在这里声明一层map,把第一层geo配置全部给到这里，
     *  纹理贴图需要设置到areaColor那里
     *  第一层的borderWidth 大于 这里的borderWidth 即可实现最外层和内部的边框样式不同的效果
     */
    // series: [
    //   {
    //     type: 'map',
    //     map: 'shandong',
    //     itemStyle: {
    //       borderWidth: 1,
    //       areaColor: document.getElementById('sticker'),
    //     },
    //   }
    // ]
  };
  chartInstance.value?.setOption(echartsOptions);
  // 重绘一次，不然省会城市会缺失，暂时没有发现BUG问题所在
  chartInstance.value?.resize();
};

onMounted(() => {
  chartInstance.value = echarts.init(map.value);
  // 监听新的series数据，并渲染地图
  watch(
    () => props.series,
    (newValue) => {
      registerMap(newValue);
    },
    {
      immediate: true,
    }
  );
  // 监听新的地图json数据，渲染地图
  watch(
    () => props.json,
    (newValue) => {
      registerMap(newValue);
    }
  );
  // chartInstance.value?.on('georoam', (params: any) => {
  //   const options: any = chartInstance.value?.getOption();
  //   if (params?.zoom) {
  //     // 缩放
  //     options.geo.slice(1)?.forEach((item: any) => {
  //       item.zoom = options?.geo?.[0]['zoom'];
  //       item.center = options?.geo?.[0]['center'];
  //     })
  //   } else {
  //     // 移动
  //     options.geo.slice(1)?.forEach((item: any) => {
  //       item.center = options?.geo?.[0]['center'];
  //     })
  //   }
  //   chartInstance.value?.setOption(options, {
  //     replaceMerge: ['geo'],
  //   });
  // })
});
</script>

<style lang="less" scoped>
.map-container {
  width: 600px; // 为了演示效果，这里固定宽高, 通常给100%, 宽度由父级DOM决定
}
</style>
