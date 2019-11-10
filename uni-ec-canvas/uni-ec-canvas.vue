<template>
    <canvas
        class="ec-canvas"
        :canvas-id="canvasId"
        @init="init"
        @touchstart="touchStart"
        @touchmove="touchMove"
        @touchend="touchEnd"
    >
</canvas>
</template>

<script>
import WxCanvas from './wx-canvas'
import * as echarts from './echarts'

let ctx
function wrapTouch (event) {
  for (let i = 0; i < event.touches.length; ++i) {
    const touch = event.touches[i]
    touch.offsetX = touch.x
    touch.offsetY = touch.y
  }
  return event
}

export default {
  props: {
    canvasId: {
      type: String,
      default: () => {
        return 'ec-canvas'
      }
    },
    ec: {
      type: Object
    }
  },
  data () {
    return {
      $curChart: {}
    }
  },
  watch: {
    'ec.option': {
      deep: true,
      handler (val, oldVal) {
        this.$curChart.setOption(val)
      }
    }
  },
  onReady: function () {
    if (!this.ec) {
      console.warn('组件需绑定 ec 变量，例：<ec-canvas id="mychart-dom-bar" ' +
        'canvas-id="mychart-bar" ec="{{ ec }}"></ec-canvas>')
      return
    }

    if (!this.ec.lazyLoad) {
      this.init()
    }
  },

  methods: {
    init (callback) {
      const version = wx.version.version.split('.').map(n => parseInt(n, 10))
      const isValid = version[0] > 1 || (version[0] === 1 && version[1] > 9) ||
        (version[0] === 1 && version[1] === 9 && version[2] >= 91)
      if (!isValid) {
        console.error('微信基础库版本过低，需大于等于 1.9.91。' +
          '参见：https://github.com/ecomfe/echarts-for-weixin' +
          '#%E5%BE%AE%E4%BF%A1%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82')
        return
      }

      ctx = wx.createCanvasContext(this.canvasId, this)

      const canvas = new WxCanvas(ctx, this.canvasId)

      echarts.setCanvasCreator(() => {
        return canvas
      })

      var query = wx.createSelectorQuery().in(this)
      query
        .select('.ec-canvas')
        .boundingClientRect(res => {
          let chart
          if (typeof callback === 'function') {
            chart = callback(canvas, res.width, res.height)
          } else if (this.ec) {
            chart = this.initChart(canvas, res.width, res.height)
          } else {
            this.triggerEvent('init', {
              canvas: canvas,
              width: res.width,
              height: res.height
            })
          }
          // delete chart._dom
          try {
            this.$curChart = chart
          } catch (e) {

          }
        }).exec()
    },

    canvasToTempFilePath (opt) {
      if (!opt.canvasId) {
        opt.canvasId = this.canvasId
      }

      ctx.draw(true, () => {
        wx.canvasToTempFilePath(opt, this)
      })
    },

    touchStart (e) {
      if (this.$curChart && e.touches.length > 0) {
        var touch = e.touches[0]
        var handler = this.$curChart.getZr().handler
        handler.dispatch('mousedown', {
          zrX: touch.x,
          zrY: touch.y
        })
        handler.dispatch('mousemove', {
          zrX: touch.x,
          zrY: touch.y
        })
        handler.processGesture(wrapTouch(e), 'start')
      }
    },

    touchMove (e) {
      if (this.$curChart && e.touches.length > 0) {
        var touch = e.touches[0]
        var handler = this.$curChart.getZr().handler
        handler.dispatch('mousemove', {
          zrX: touch.x,
          zrY: touch.y
        })
        handler.processGesture(wrapTouch(e), 'change')
      }
    },

    touchEnd (e) {
      if (this.$curChart) {
        const touch = e.changedTouches ? e.changedTouches[0] : {}
        var handler = this.$curChart.getZr().handler
        handler.dispatch('mouseup', {
          zrX: touch.x,
          zrY: touch.y
        })
        handler.dispatch('click', {
          zrX: touch.x,
          zrY: touch.y
        })
        handler.processGesture(wrapTouch(e), 'end')
      }
    },
    initChart (canvas, width, height) {
      const chart = echarts.init(canvas, null, {
        width: width,
        height: height
      })
      canvas.setChart(chart)
      chart.setOption(this.ec.option)
      return chart
    }
  }
}

</script>

<style lang="scss">
.ec-canvas {
  width: 100%;
  height: 100%;
  display:block;
}
</style>
