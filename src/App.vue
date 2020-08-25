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
.zoom {
  color: #FFF;
  font-size: 30px;
}
</style>

<template>
  <div class="page" flex="dir:top main:center cross:center">
    <p class="zoom">isZoom: {{ isZoom }}</p>
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

function yearStartUnix (year) {
  return moment(`${year}-01-01`).unix()
}

export default {
  data () {
    return {
      source,
      chart: null,
      isZoom: false
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
            xAxisIndex: [0],
            minSpan: 10
          }
        ],
        tooltip: {
          trigger: 'axis',
          formatter: function (params) {
            return params.map(param => {
              const start = yearStartUnix(param.seriesName)
              const axisValue = start + param.axisValue
              const label = moment.unix(axisValue).format('MM/DD')
              return `${param.seriesName}/${label} ${Number(param.data[1])}元`
            }).join('<br/>')
          },
          axisPointer: {
            animation: false
          }
        },
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
        xAxis: {
          type: 'value',
          maxInterval: 60 * 60 * 24 * 31,
          minInterval: 60 * 60 * 24,
          max: 60 * 60 * 24 * 366,
          splitLine: {
            show: false
          },
          axisLabel: {
            color: '#333333',
            fontSize: 10,
            formatter: (value, index) => {
              const date = moment.unix(yearStartUnix('2020') + value)
              if (value === 60 * 60 * 24 * 366) return ''
              return this.isZoom ? date.format('M月D日') : date.format('M月')
            }
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
        series: data.map((serie, serieIndex) => {
          const start = yearStartUnix(serie.name)
          return {
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
            data: serie.data.map(e => [
              e[0] - start,
              e[1]
            ])
          }
        })
      }
      console.log(option)
      // 使用刚指定的配置项和数据显示图表。
      this.chart.setOption(option, true)
    },
    chartInit () {
      this.chart = echarts.init(document.getElementById('chart'))
      this.chart.on('dataZoom', params => {
        this.isZoom = params.batch.reduce((result, item) => result || (item.end - item.start) < 50, false)
      })
    }
  }
}
</script>
