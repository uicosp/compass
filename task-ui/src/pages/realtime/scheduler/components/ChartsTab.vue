<script setup lang="ts">
import * as echarts from 'echarts'
const props = defineProps({
  data: {
    type: Array,
    default: () => [],
  },
  unit: {
    type: String,
    default: '',
  },
})
const emit = defineEmits(['tabChange'])
const activeName: string = $ref('memory')
const tabList = [
  { label: '内存趋势', name: 'memory' },
  { label: 'CPU趋势', name: 'cpu' },
  { label: '数量趋势', name: 'num' },
]
const option = {
  xAxis: {
    type: 'category',
    data: [],
  },
  yAxis: {
    type: 'value',
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '10%',
    containLabel: true,
  },
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'shadow',
    },
  },
  legend: {},
  series: [
    {
      name: '可优化量',
      data: [],
      type: 'bar',
      itemStyle: {
        normal: {
          color: '#90EE90',
        },
      },
    },
    {
      name: '总使用量',
      data: [],
      type: 'bar',
      itemStyle: {
        normal: {
          color: '#00a698',
        },
      },
    },
  ],
}
let chart: any = null
const handleClick = (val) => {
  emit('tabChange', val.paneName)
}
watch(
  () => props.data,
  () => {
    console.log(props.data)
    const xData = props.data.jobUsage.data.map(item => item.date.slice(0, 10))
    const yData1 = props.data.jobUsage.data.map(item => item.count)
    const yData2 = props.data.totalUsage.data.map(item => item.count)
    option.xAxis.data = xData
    option.yAxis.name = props.data.unit ? `单位（${props.data.unit}）` : ''
    option.series[0].data = yData1
    option.series[1].data = yData2
    option.series[0].name = props.data.jobUsage.name
    option.series[1].name = props.data.totalUsage.name
    chart.setOption(option)
  },
)
let resizeObserver: any = null
const init = () => {
  const currentInstance: any = getCurrentInstance()
  chart = echarts.init(currentInstance.proxy.$refs.test)
  chart.setOption(option)
  resizeObserver = new ResizeObserver((entries) => {
    chart.resize()
  })
  resizeObserver.observe(currentInstance.proxy.$refs.test)
}
onMounted(() => {
  init()
})
onBeforeUnmount(() => {
  resizeObserver.disconnect()
})
</script>

<template>
  <el-tabs v-model="activeName" @tab-click="handleClick">
    <el-tab-pane v-for="item in tabList" :key="item.name" :label="item.label" :name="item.name" />
    <div ref="test" style="height:250px;width:100%" />
  </el-tabs>
</template>

<style lang="scss" scoped>
:deep(.el-tabs__nav-wrap){
  display: flex;
  justify-content: center;
}
:deep(.el-tabs__nav-scroll){
  width:250px;
}
</style>
