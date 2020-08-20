<style lang="scss">
html, body, .page {
  background-color: #24292E;
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
}
#chart {
  height: 300px;
  width: 400px;
  background-color: #FFF;
  border-radius: 2px;
}
</style>

<template>
  <div class="page" flex="main:center cross:center">
    <div id="chart"></div>
  </div>
</template>

<script>
import moment from 'moment'
import echarts from 'echarts/lib/echarts'
import 'echarts/lib/chart/line'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/component/title'
import 'echarts/lib/component/legend'
import 'echarts/lib/component/legendScroll'
import 'echarts/lib/component/dataZoom'

import source from './source'

const daysInMonth = [31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
const days = []

for (let month = 0; month < 12; month++) {
  const daysCount = daysInMonth[month]
  for (let day = 1; day <= daysCount; day++) {
    days.push(`${month + 1}/${day}`)
  }
}

export default {
  data () {
    return {
      source,
      chart: null
    }
  },
  mounted () {
    this.chartInit()
    this.chartUpdate(this.source)
  },
  methods: {
    chartUpdate (data = []) {
      if (!this.chart) {
        this.chartInit()
      }
      // 指定图表的配置项和数据
      var option = {
        dataZoom: [
          {
            type: 'inside',
            xAxisIndex: [0]
          },
          {
            type: 'inside',
            yAxisIndex: [0]
          }
        ],
        legend: {
          icon: 'circle',
          itemWidth: 6,
          itemHeight: 6,
          type: 'scroll',
          bottom: 10,
          left: 10,
          right: 10,
          pageIconSize: 10,
          data: data.map((serie, serieIndex) => ({
            name: serie.name
          }))
        },
        grid: {
          left: 10,
          right: 10,
          bottom: 40,
          top: 40,
          containLabel: true
        },
        title: {
          left: 'center',
          top: 10,
          text: '价格趋势图',
          textStyle: {
            color: '#333333',
            fontWeight: 500,
            fontSize: 16
          }
        },
        tooltip: {
          trigger: 'axis',
          formatter: function (params) {
            return params.map(param => `${param.seriesName} ${param.axisValue} ${param.data[1]}`).join('<br/>')
          },
          axisPointer: {
            animation: false
          }
        },
        xAxis: {
          data: days,
          splitLine: {
            show: false
          },
          axisLabel: {
            color: '#333333',
            fontSize: 10
          },
          axisLine: {
            lineStyle: {
              color: '#E8E8E8'
            }
          }
        },
        yAxis: {
          splitLine: {
            show: false
          },
          axisLabel: {
            color: '#333333',
            fontSize: 10
          },
          axisLine: {
            lineStyle: {
              color: '#E8E8E8'
            }
          }
        },
        series: data.map((serie, serieIndex) => ({
          name: serie.name,
          type: 'line',
          smooth: 0.2,
          lineStyle: {
            color: ['#FF9A35', '#5584C0', '#B2B2B2', '#FFBC2B', '#26BFD1'][serieIndex]
          },
          itemStyle: {
            color: ['#FF9A35', '#5584C0', '#B2B2B2', '#FFBC2B', '#26BFD1'][serieIndex],
            borderWidth: 0
          },
          symbol: 'circle',
          symbolSize: 2,
          data: serie.data.map(e => {
            const date = moment.unix(e[0]).format('M/D')
            return [date, e[1]]
          })
        }))
      }
      console.log(option)
      // 使用刚指定的配置项和数据显示图表。
      this.chart.setOption(option, true)
    },
    chartInit () {
      this.chart = echarts.init(document.getElementById('chart'))
    }
  }
}
</script>
