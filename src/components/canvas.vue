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
  data () {
    return {
      canvas: '', // fabricCanvas
      historyLock: false, // 前进后退列表锁
      undoHistory: [], // 后退历史列表
      redoHistroy: [] // 前进历史列表
    }
  },
  mounted () {
    this.canvasPorpertyInit()

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
      console.log(this.undoHistory)
      if (this.undoHistory.length > 0) {
        this.historyLock = true // 执行前先锁定列表
        // undo弹出末尾状态，redo压入该状态
        if (this.undoHistory.length > 1) {
          // 最开始的白纸状态不进入redo列表
          this.redoHistroy.push(this.undoHistory.pop())
        }
        // 取弹出状态后的undo的最后一位（即弹出前的倒数第二为），重新渲染该状态，并取消锁定
        const content = this.undoHistory[this.undoHistory.length - 1]
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
      console.log(this.redoHistroy)
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
    switchMode () {
      this.canvas.isDrawingMode = !this.canvas.isDrawingMode
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
      console.log(e)
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
