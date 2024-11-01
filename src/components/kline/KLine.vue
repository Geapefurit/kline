<template>
  <div id='chart-container' />
</template>
<style>
* {
  margin: 0;
  padding: 0;
}
#chart-container {
  position: relative;
  height: 100vh;
  display: block;
  overflow: hidden;
}
</style>

<script setup lang='ts'>
import axios from 'axios'
import { onMounted } from 'vue'
import * as echarts from 'echarts/core'
import {
  ToolboxComponent,
  ToolboxComponentOption,
  TooltipComponent,
  TooltipComponentOption,
  GridComponent,
  GridComponentOption,
  VisualMapComponent,
  VisualMapComponentOption,
  LegendComponent,
  LegendComponentOption,
  BrushComponent,
  BrushComponentOption,
  DataZoomComponent,
  DataZoomComponentOption
} from 'echarts/components'
import {
  CandlestickChart,
  CandlestickSeriesOption,
  LineChart,
  LineSeriesOption,
  BarChart,
  BarSeriesOption
} from 'echarts/charts'
import { UniversalTransition } from 'echarts/features'
import { CanvasRenderer } from 'echarts/renderers'

echarts.use([
  ToolboxComponent,
  TooltipComponent,
  GridComponent,
  VisualMapComponent,
  LegendComponent,
  BrushComponent,
  DataZoomComponent,
  CandlestickChart,
  LineChart,
  BarChart,
  CanvasRenderer,
  UniversalTransition
])

type EChartsOption = echarts.ComposeOption<
  | ToolboxComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | VisualMapComponentOption
  | LegendComponentOption
  | BrushComponentOption
  | DataZoomComponentOption
  | CandlestickSeriesOption
  | LineSeriesOption
  | BarSeriesOption
>

interface KPoint {
  Nums: number[];
  Times: number[];
}

interface KPointsForLine {
  KPointType: string;
  KPoints: KPoint[];
  Limit: number;
  Offset: number;
  OriginalTime: number;
  TokenPairID: number;
  Total: number;
}

onMounted(() => {
  const chartDom = document.getElementById('chart-container') as HTMLElement
  const myChart = echarts.init(chartDom)

  const splitData = (info: KPointsForLine) => {
    const rawDatas = info.KPoints
    const categoryData = []
    const values = []

    for (let i = 0; i < rawDatas.length; i++) {
      categoryData.push(rawDatas[i].Times[0].toString())
      values.push(rawDatas[i].Nums)
    }

    return {
      categoryData: categoryData,
      values: values
    }
  }

  const calculateMA = (dayCount: number, data: { values: number[][] }) => {
    const result = []
    for (let i = 0, len = data.values.length; i < len; i++) {
      if (i < dayCount) {
        result.push('-')
        continue
      }
      let sum = 0
      for (let j = 0; j < dayCount; j++) {
        sum += data.values[i - j][1]
      }
      result.push(+(sum / dayCount).toFixed(3))
    }
    return result
  }

  const calculateZoomStart = (itemsLen: number) => {
    let result = 90
    if (itemsLen < 50) {
      result = 0
    } else {
      result = 100 - 50 / itemsLen * 100
    }
    return result
  }

  axios.post('http://172.16.31.48:30100/v1/get/kpoints/for/line', {
    KPointType: 'FiveSecond',
    Limit: -100,
    Offset: 0,
    OriginalTime: 0,
    TokenPairID: 1
  }).then((response) => {
    const data = splitData(response.data as KPointsForLine)
    const option: EChartsOption = {
      animation: true,
      legend: {
        bottom: 20,
        left: 'center',
        data: ['Dow-Jones index', 'MA5', 'MA10', 'MA20', 'MA30']
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'cross'
        },
        borderWidth: 1,
        borderColor: '#ccc',
        padding: 10,
        textStyle: {
          color: '#000'
        }
      },
      axisPointer: {
        link: [
          {
            xAxisIndex: 'all'
          }
        ],
        label: {
          backgroundColor: '#777'
        }
      },
      toolbox: {
        feature: {
          dataZoom: {
            yAxisIndex: false
          },
          brush: {
            type: ['lineX', 'clear']
          }
        }
      },
      brush: {
        xAxisIndex: 'all',
        brushLink: 'all',
        outOfBrush: {
          colorAlpha: 0.1
        }
      },
      visualMap: {
        show: false,
        seriesIndex: 5,
        dimension: 2,
        pieces: [
          {
            value: 1,
            color: '#ec0000'
          },
          {
            value: -1,
            color: '#00da3c'
          }
        ]
      },
      grid: [
        {
          left: '10%',
          right: '8%',
          height: '70%'
        }
      ],
      xAxis: [
        {
          type: 'category',
          data: data.categoryData,
          boundaryGap: false,
          axisLine: { onZero: false },
          splitLine: { show: false },
          min: 'dataMin',
          max: 'dataMax',
          axisPointer: {
            z: 100
          }
        }
      ],
      yAxis: [
        {
          scale: true,
          splitArea: {
            show: true
          }
        }
      ],
      dataZoom: [
        {
          type: 'inside',
          xAxisIndex: [0],
          start: calculateZoomStart(data.categoryData.length),
          end: 100
        },
        {
          show: true,
          xAxisIndex: [0],
          type: 'slider',
          top: '85%',
          start: calculateZoomStart(data.categoryData.length),
          end: 100
        }
      ],
      series: [
        {
          name: 'Dow-Jones index',
          type: 'candlestick',
          data: data.values,
          itemStyle: {
            color: '#00da3c',
            color0: '#ec0000',
            borderColor: undefined,
            borderColor0: undefined
          }
        },
        {
          name: 'MA5',
          type: 'line',
          data: calculateMA(5, data),
          smooth: true,
          lineStyle: {
            opacity: 0.5
          }
        },
        {
          name: 'MA10',
          type: 'line',
          data: calculateMA(10, data),
          smooth: true,
          lineStyle: {
            opacity: 0.5
          }
        },
        {
          name: 'MA20',
          type: 'line',
          data: calculateMA(20, data),
          smooth: true,
          lineStyle: {
            opacity: 0.5
          }
        },
        {
          name: 'MA30',
          type: 'line',
          data: calculateMA(30, data),
          smooth: true,
          lineStyle: {
            opacity: 0.5
          }
        }
      ]
    }

    myChart.setOption(
      option,
      true
    )

    if (data.categoryData.length > 20) {
      myChart.dispatchAction({
        type: 'brush',
        areas: [
          {
            brushType: 'lineX',
            coordRange: [data.categoryData[data.categoryData.length - 20], data.categoryData[data.categoryData.length - 1]],
            xAxisIndex: 0
          }
        ]
      })
    }
  }
  ).catch(function (error) { // 请求失败处理
    console.log(error)
  })
})
</script>
