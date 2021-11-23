<template>
  <div ref="canvasContainer" class="canvasContainer">
    <canvas ref="canvas" id="origin-canvas"></canvas>
    <input
      ref="imageInputBtn"
      style="display: none"
      id="imageInputBtn"
      type="file"
      @change="imageAdd($event)"
    />
    <input
      ref="jsonInputBtn"
      style="display:none"
      id="jsonInputBtn"
      type="file"
      @change="jsonAdd($event)"
    />
  </div>
</template>

<script>
// @ is an alias to /src
import { fabric } from 'fabric'

export default {
  name: 'Home',
  components: {},
  props: {
    lineWidth: {
      type: Number,
      default: 1
    },
    lineColor: {
      type: String,
      default: '#000000'
    }
  },
  data () {
    return {
      canvas: '', // fabricCanvas
      historyLock: false, // 前进后退列表锁
      undoHistory: [], // 后退历史列表
      redoHistroy: [], // 前进历史列表,
      vLinePatternBrush: '' // 测试笔刷
    }
  },
  mounted () {
    this.canvasPorpertyInit()

    /* 创建笔刷 */
    this.createBrush()
    // if (fabric.PatternBrush) {
    //   this.vLinePatternBrush = new fabric.PatternBrush(this.canvas)
    //   this.vLinePatternBrush.getPatternSrc = function () {
    //     var patternCanvas = fabric.document.createElement('canvas')
    //     patternCanvas.width = patternCanvas.height = 10
    //     var ctx = patternCanvas.getContext('2d')

    //     ctx.strokeStyle = this.color
    //     ctx.lineWidth = 5
    //     ctx.beginPath()
    //     ctx.moveTo(0, 5)
    //     ctx.lineTo(10, 5)
    //     ctx.closePath()
    //     ctx.stroke()

    //     return patternCanvas
    //   }
    // }
    /*
    监听屏幕大小变化,旋转
    */
    window.addEventListener('resize', function () {
      // this.canvasWidthHeightInit()
    })

    window.addEventListener(
      'orientationchange',
      function () {
        setTimeout(() => {
          this.canvasWidthHeightInit()
        }, 200)
      }.bind(this)
    )
    /*
      删除功能相关
      (如果日后与放大缩小功能有冲突可以加一个删除的标志标志变量)
    */
    this.canvas.on(
      'selection:created',
      function (e) {
        this.historyLock = true // 删除时锁定列表、
        if (e.target._objects) {
          // 多选删除
          const etCount = e.target._objects.length
          for (let etindex = 0; etindex < etCount; etindex++) {
            this.canvas.remove(e.target._objects[etindex])
          }
          // 删除完之后undo写入当前状态
          this.undoHistory.push(JSON.stringify(this.canvas)) // UNDO処理
          this.historyLock = false
        } else {
          // 单选删除
          this.canvas.remove(e.target)
          // 删除完之后undo写入当前状态
          this.undoHistory.push(JSON.stringify(this.canvas)) // UNDO処理
          this.historyLock = false
        }
        this.canvas.discardActiveObject() // 清楚选中框
      }.bind(this)
    )
    /*
      前进后退功能相关
    */
    // 一开始把当前状态写入undo历史列表
    this.undoHistory.push(JSON.stringify(this.canvas))
    // 监听canvas对象添加事件 向undo列表写入当前状态
    this.canvas.on(
      'object:added',
      function () {
        // 锁定则中止
        if (!this.historyLock) {
          this.undoHistory.push(JSON.stringify(this.canvas))
          this.redoHistroy.length = 0 // 清空redo列表 防止此时执行redo操作重复绘制？
        }
      }.bind(this)
    )
    // 监听canvas对象修改事件 同上
    this.canvas.on(
      'object:modified',
      function () {
        if (!this.historyLock) {
          this.undoHistory.push(JSON.stringify(this.canvas))
          this.redoHistroy.length = 0
        }
      }.bind(this)
    )
  },
  methods: {
    /* 创建笔刷 */
    createBrush () {
      fabric.MarkerBrush = fabric.util.createClass(fabric.BaseBrush, {
        color: '#000', // 颜色
        opacity: 1, // 透明度
        width: 30, // 宽度

        _baseWidth: 10, // 基础宽度
        _lastPoint: null,
        _lineWidth: 3,
        _point: null,
        _size: 0,

        initialize: function (canvas, opt) {
          opt = opt || {}

          this.canvas = canvas
          this.width = opt.width || canvas.freeDrawingBrush.width
          this.color = opt.color || canvas.freeDrawingBrush.color
          this.opacity = opt.opacity || canvas.contextTop.globalAlpha
          this.canvas.contextTop.globalAlpha = this.opacity
          this._point = new fabric.Point()

          this.canvas.contextTop.lineJoin = 'round'
          this.canvas.contextTop.lineCap = 'round'
        },

        _render: function (pointer) {
          var ctx, lineWidthDiff

          ctx = this.canvas.contextTop

          ctx.beginPath()

          for (
            let i = 0, len = this._size / this._lineWidth / 2;
            i < len;
            i++
          ) {
            lineWidthDiff = (this._lineWidth - 1) * i

            ctx.globalAlpha = 0.8 * this.opacity
            ctx.moveTo(
              this._lastPoint.x + lineWidthDiff,
              this._lastPoint.y + lineWidthDiff
            )
            ctx.lineTo(pointer.x + lineWidthDiff, pointer.y + lineWidthDiff)
            ctx.stroke()
          }

          this._lastPoint = new fabric.Point(pointer.x, pointer.y)
        },

        onMouseDown: function (pointer) {
          this._lastPoint = pointer
          this.canvas.contextTop.strokeStyle = this.color
          this.canvas.contextTop.lineWidth = this._lineWidth
          this._size = this.width + this._baseWidth
        },

        onMouseMove: function (pointer) {
          if (this.canvas._isCurrentlyDrawing) {
            this._render(pointer)
          }
        },

        onMouseUp: function () {
          this.canvas.contextTop.globalAlpha = this.opacity
          this.canvas.contextTop.globalAlpha = 1
        }
      })
    },
    /* 保存当前canvas为json并下载 */
    saveTraceAsJSON () {
      const content = JSON.stringify(this.canvas)
      const blob = new Blob([content])
      const a = document.createElement('a')
      a.download = '签名.json'
      a.href = URL.createObjectURL(blob)
      a.click()
      // 这里可以emit事件向外传递json文件
    },
    /* 后退函数 */
    undo () {
      // console.log(this.undoHistory)
      if (this.undoHistory.length > 0) {
        this.historyLock = true // 执行前先锁定列表
        // undo弹出末尾状态，redo压入该状态
        if (this.undoHistory.length > 1) {
          // 最开始的白纸状态不进入redo列表
          this.redoHistroy.push(this.undoHistory.pop())
        }
        // 取弹出状态后的undo的最后一位（即弹出前的倒数第二为），重新渲染该状态，并取消锁定
        const content = this.undoHistory[this.undoHistory.length - 1]
        // console.log(content)
        this.canvas.loadFromJSON(
          content,
          function () {
            this.canvas.renderAll()
            this.historyLock = false
          }.bind(this)
        )
      }
    },
    /* 前进函数 */
    redo () {
      // console.log(this.redoHistroy)
      if (this.redoHistroy.length > 0) {
        this.historyLock = true // 执行前锁定列表
        // redo弹出末尾状态 undo压入 canvas渲染该状态
        const content = this.redoHistroy.pop()
        this.undoHistory.push(content)
        this.canvas.loadFromJSON(
          content,
          function () {
            this.canvas.renderAll()
            this.historyLock = false
          }.bind(this)
        )
      }
    },
    /* 初始化Canvas */
    canvasPorpertyInit () {
      const ref = this.$refs.canvas // 初始canvas元素
      this.canvas = new fabric.Canvas(ref, {
        isDrawingMode: true
      }) // 初始化fabric的canvas
      this.canvasWidthHeightInit()
      //   const rect = new fabric.Rect({
      //     fill: 'red',
      //     width: 100,
      //     height: 100
      //   })
      //   this.canvas.add(rect)
    },
    /*
    初始化canvas的宽高
    设置为容器百分百大小
    */
    canvasWidthHeightInit () {
      const canvasContainer = this.$refs.canvasContainer // 容器
      const clientHeight = canvasContainer.clientHeight
      const clientWidth = canvasContainer.clientWidth
      //   console.log(clientWidth, clientHeight)
      this.canvas.setDimensions({
        width: clientWidth,
        height: clientHeight
      })
    },
    /* 绘画模式切换 */
    switchMode (val) {
      this.canvas.isDrawingMode = val
    },
    /* 清除画布 */
    canvasClear () {
      this.historyLock = true
      this.canvas.clear()
      this.undoHistory.push(JSON.stringify(this.canvas)) // 每做一次清除操作都会有一个空白状态进入undo与redo列表的循环
      this.historyLock = false
    },
    /* 下载画布内容 */
    download (type) {
      if (type === 'svg') {
        const svg = this.canvas.toSVG() // 获取svg
        const a = document.createElement('a')
        const blob = new Blob([svg], { type: 'image/svg+xml' }) // 利用svg创建blob对象
        const blobURL = URL.createObjectURL(blob) // 利用blob对象创建对应的blobURL
        a.href = blobURL
        a.download = 'signature.svg'
        a.click()
      } else {
        // png or jpg
        const ext = type
        const base64 = this.canvas.toDataURL({
          format: ext,
          enableRetinaScaling: true
        })
        const link = document.createElement('a')
        link.href = base64
        link.download = `signature.${ext}`
        link.click()
      }
    },
    /* 图片导入触发点击事件 */
    imageInput () {
      this.$refs.imageInputBtn.click()
    },
    /* json导入触发点击 */
    jsonInput () {
      this.$refs.jsonInputBtn.click()
    },
    /* 添加图片至canvas */
    imageAdd (e) {
      const that = this
      const fileType = e.target.files[0].type
      const url = URL.createObjectURL(e.target.files[0])
      console.log(that.canvas)

      if (fileType === 'image/png' || fileType === 'image/jpeg') {
        fabric.Image.fromURL(url, function (img) {
          that.canvas.add(img)
        })
      } else if (fileType === 'image/svg+xml') {
        fabric.loadSVGFromURL(url, function (objects, options) {
          const svg = fabric.util.groupSVGElements(objects, options)
          that.canvas.add(svg)
        })
      } else {
        alert('不支持此类型文件')
      }
    },
    /* 导入json至canvas */
    jsonAdd (e) {
      // console.log(e)
      const that = this
      const file = e.target.files[0]
      const fileType = e.target.files[0].type
      // console.log(e.target.files[0])
      // console.log(that.canvas)

      if (fileType === 'application/json') {
        let jsonFile = ''
        const fileReader = new FileReader()
        // 读取内容并加载在画板上
        fileReader.onload = function (evt) {
          jsonFile = evt.target.result
          //   console.log(jsonFile);
          that.canvas.loadFromJSON(jsonFile, function () {
            that.canvas.renderAll()
          })
        }
        fileReader.readAsText(file)
      } else {
        alert('不支持此类型文件')
      }
    },
    /* 设置笔触粗细 */
    setLineWidth () {
      this.canvas.freeDrawingBrush.width = this.lineWidth
    },
    /* 设置笔触颜色 */
    setLineColor () {
      // console.log(this.lineColor);
      this.canvas.freeDrawingBrush.color = this.lineColor
      const brush = this.canvas.freeDrawingBrush
      if (brush.getPatternSrc) {
        brush.source = brush.getPatternSrc()
      }
    },
    /* 笔锋测试 */
    setPenShape () {
      // this.canvas.freeDrawingBrush = this.vLinePatternBrush
      // const brush = this.canvas.freeDrawingBrush
      // if (brush.getPatternSrc) {
      //   brush.source = brush.getPatternSrc()
      // }
      this.canvas.freeDrawingBrush = new fabric.MarkerBrush(this.canvas, {})
    }
  }
}
</script>

<style scoped>
.canvasContainer {
  width: 100%;
  height: 100%;
}
</style>
