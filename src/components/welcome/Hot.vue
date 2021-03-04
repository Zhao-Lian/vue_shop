<template>
  <div>
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main2" style="width: 100%; height: 380px"></div>
  </div>
</template>
<script>
import echarts from 'echarts'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 1
      },
      mychart: {},
      option: {
        title: {
          text: '销量趋势',
          textStyle: {
            fontSize: 20
          }
        },
        grid: {
          top: 70
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#999'
            }
          }
        },
        legend: {
          data: ['销量', '增长率'],
          top: 20
        },
        xAxis: [
          {
            type: 'category',
            data: (function() {
              let now = new Date()
              const res = []
              let len = 7
              while (len--) {
                res.unshift(now.getDate() + '日')
                now = new Date(now - 24 * 60 * 60 * 1000)
              }
              return res
            })(),
            axisPointer: {
              type: 'shadow'
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            name: '销量',
            // min: 0,
            // max: 250,
            // interval: 50,
            axisLabel: {
              formatter: '{value}'
            }
          },
          {
            type: 'value',
            name: '增长率',
            min: -30,
            // max: 25,
            // interval: 5,
            axisLabel: {
              formatter: '{value} %'
            },
            splitLine: {
              show: false
            }
          }
        ],
        series: [
          {
            name: '销量',
            type: 'bar',
            barWidth: '60%',
            data: [10, 13, 17, 22, 20, 25, 27]
          },
          {
            name: '增长率',
            type: 'line',
            yAxisIndex: 1,
            data: []
          }
        ],
        color: [
          '#5470c6',
          '#91cc75',
          '#fac858',
          '#ee6666',
          '#73c0de',
          '#3ba272',
          '#fc8452',
          '#9a60b4',
          '#ea7ccc'
        ]
      }
    }
  },
  created() {
    const list = this.option.series[0].data
    const res = []
    for (let i = 0; i < 6; i++) {
      res.push((((list[i + 1] - list[i]) / list[i]) * 100).toFixed(2))
    }
    res.unshift('-')
    this.option.series[1].data = res
  },
  mounted() {
    // 基于准备好的dom，初始化echarts实例
    this.myChart = echarts.init(document.getElementById('main2'))

    // 使用刚指定的配置项和数据显示图表。
    this.myChart.setOption(this.option)

    window.addEventListener('resize', () => {
      this.myChart.resize()
    })
  }
}
</script>
<style lang="less" scoped>
#main2 {
  margin-top: 10px;
}
</style>
