<template>
  <view>
    <uni-ec-canvas
      class="uni-ec-canvas"
      id="line-chart"
      ref="canvas"
      canvas-id="lazy-load-chart"
      :ec="ec"
      @inited="inited"
    >
    </uni-ec-canvas>
  </view>
</template>

<script>
import uniEcCanvas from '@/components/uni-ec-canvas/uni-ec-canvas'
export default {
  data() {
    return {
      ec: {
        lazyLoad: true,
        option: {},
      },
    }
  },
  methods: {
    inited(ins) {
      const option = {
        xAxis: {
          type: 'time',
          axisLabel: {
            formatter: function (value, index) {
              // 格式化成月/日，只在第一个刻度显示年份
              var date = new Date(value)
              var texts = [date.getMonth() + 1, date.getDate()]
              if (index === 0) {
                texts.unshift(date.getYear())
              }
              return texts.join('/')
            },
            // formatter: {
            // 	year: '{yearStyle|{yyyy}}\n{monthStyle|{MMM}}',
            // 	month: '{monthStyle|{MMM}}'
            // }
            // formatter: {
            // 	month: '{MMMM}', // 一月、二月、……
            // 	day: '{d}日' // 1日、2日、……
            // }
          },
        },
        yAxis: {
          type: 'value',
        },
        series: [
          {
            data: [
              ['2021-11-01 08:00:00', 20],
              ['2021-11-01 12:00:00', 30],
              ['2021-11-01 15:00:00', 30],
              ['2021-11-02 08:00:00', 40],
              ['2021-11-03 08:00:00', 30],
              ['2021-11-04 08:00:00', 10],
            ],
            type: 'line',
          },
          {
            data: [
              ['2021-11-01 00:00:00', 0],
              ['2021-11-05 00:00:00', 0],
            ],
            type: 'line',
          },
        ],
      }

      ins.setOption(option)
    },
  },
  onReady() {
    this.$refs['canvas'].init()
  },
  components: {
    uniEcCanvas,
  },
}
</script>
<style scoped>
.uni-ec-canvas {
  width: 750upx;
  height: 750upx;
  display: block;
}
</style>
