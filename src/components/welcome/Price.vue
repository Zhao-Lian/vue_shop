<template>
  <div>
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main" style="width: 100%; height: 360px"></div>
  </div>
</template>
<script>
import echarts from 'echarts'
export default {
  props: ['price'],
  data() {
    return {
      mychart: {},
      option: {
        title: {
          text: '成交金额分布（元）',
          textStyle: {
            fontSize: 20
          }
        },
        tooltip: { trigger: 'item' },
        legend: {
          orient: 'vertical',
          left: 'left',
          top: 'center'
        },
        series: [
          {
            name: '销量',
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            label: {
              show: false,
              position: 'center'
            },
            emphasis: {
              label: {
                show: true,
                fontSize: '35',
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            },
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
  mounted() {
    // 基于准备好的dom，初始化echarts实例
    this.myChart = echarts.init(document.getElementById('main'))

    // 使用刚指定的配置项和数据显示图表。
    // this.changePricePie()

    window.addEventListener('resize', () => {
      this.myChart.resize()
    })
  },
  computed: {
    // isChange() {
    //   console.log(this.price.reduce((total, item) => total + item.value, 0))
    //   return this.price.reduce((total, item) => total + item.value, 0)
    // }
  },
  watch: {
    // isChange() {
    //   this.changePricePie()
    // }
    price: {
      handler(val, oldVal) {
        this.changePricePie()
      },
      deep: true
    }
  },
  methods: {
    changePricePie() {
      this.option.series[0].data = this.price
      this.myChart.setOption(this.option)
    }
  }
}
</script>
<style lang="less" scoped>
</style>
