<template>
  <div class="container">
    <el-menu class="el-menu-demo" mode="horizontal">
      <el-menu-item class="menu-item" index="1">
        <el-button disabled @click="switchMode">{{
          Mode
        }}</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="1"
        ><el-button @click="download">保存</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="2">
        格式：
        <el-select
          class="selectModule"
          v-model="imageType"
          placeholder="请选择"
        >
          <el-option
            v-for="item in options"
            :key="item.vlaue"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-menu-item>
      <el-menu-item class="menu-item" index="3"
        ><el-button @click="clear">清除</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="8" @click="imageInput"
        ><el-button>导入图片</el-button></el-menu-item
      >
      <!-- <el-menu-item class="menu-item" index="4"
        ><el-button
          :type="btnHighlight || imgFlag ? 'primary' : ''"
          @click="repaint"
          >重绘</el-button
        ></el-menu-item
      >
      <el-menu-item class="menu-item" index="5"
        ><el-button
          :type="btnHighlight || imgFlag ? 'primary' : ''"
          @click="clearTrace"
          >清除重绘</el-button
        ></el-menu-item
      > -->
      <el-menu-item class="menu-item" index="6"
        ><el-button @click="saveTraceAsJSON">导出轨迹</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="7"
        ><el-button @click="jsonInput">导入轨迹</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="15"
        ><el-button @click="traceForward">前进</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="9"
        ><el-button @click="traceBackward">撤回</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="10"
        ><el-button @click="eraserTrigger">橡皮擦</el-button></el-menu-item
      >
      <el-menu-item class="menu-item" index="11"
        ><el-button @click="setPenShape">笔锋</el-button></el-menu-item
      >
      <!-- <el-menu-item class="menu-item" index="12">
        <el-button @click="scaleUp">放大</el-button>
      </el-menu-item>
      <el-menu-item class="menu-item" index="13">
        <el-button @click="scaleDown">缩小</el-button>
      </el-menu-item> -->
      <el-menu-item class="menu-item" index="14">
        <el-button @click="showLineWidthDialog">粗细</el-button>
      </el-menu-item>
      <el-menu-item class="menu-item" index="15">
        <span>颜色：</span>
        <el-color-picker
          @change="colorChange"
          v-model="color"
        ></el-color-picker>
      </el-menu-item>
      <!-- <el-menu-item class="menu-item" index="16">
        <span>放大倍数{{ magnifi }}</span>
      </el-menu-item> -->
    </el-menu>
    <!-- 画布组件 -->
    <fabric-canvas :lineWidth="lineWidth" :lineColor="color" ref="sig" />
    <el-dialog title="粗细" :visible.sync="lineWidthDialogVisible">
      <el-slider @change="lineWidthChange" v-model="lineWidth"></el-slider>
    </el-dialog>
  </div>
</template>

<script>
// @ is an alias to /src
import fabricCanvas from '@/components/canvas.vue'

export default {
  name: 'Home',
  components: { fabricCanvas },
  data () {
    return {
      options: [
        {
          value: 'jpg',
          label: 'jpg'
        },
        {
          value: 'png',
          label: 'png'
        },
        {
          value: 'svg',
          label: 'svg'
        }
      ],
      drawingMode: true,
      imageType: 'jpg',
      lineWidthDialogVisible: false,
      lineWidth: 5,
      color: '#000000'
    }
  },
  computed: {
    Mode () {
      return this.drawingMode ? '书写模式' : '选择模式'
    }
  },
  methods: {
    /* 绘画模式切换 */
    switchMode () {
      this.drawingMode = !this.drawingMode
      this.$refs.sig.switchMode(this.drawingMode)
    },
    /* 清除画布 */
    clear () {
      this.$refs.sig.canvasClear()
    },
    /* 保存画布 */
    download () {
      // console.log(this.imageType)
      this.$refs.sig.download(this.imageType)
    },
    /* 导入图片功能入口 */
    imageInput () {
      this.$refs.sig.imageInput()
    },
    /* 前进 */
    traceForward () {
      this.$refs.sig.redo()
    },
    /* 后退 */
    traceBackward () {
      this.$refs.sig.undo()
    },
    /* 保存为json */
    saveTraceAsJSON () {
      this.$refs.sig.saveTraceAsJSON()
    },
    /* json导入 */
    jsonInput () {
      this.$refs.sig.jsonInput()
    },
    /* 触发橡皮擦 */
    eraserTrigger () {
      this.drawingMode = !this.drawingMode
      this.$refs.sig.switchMode(this.drawingMode)
    },
    /* 展示粗细对话框 */
    showLineWidthDialog () {
      this.lineWidthDialogVisible = true
    },
    /* 改变笔触 */
    lineWidthChange () {
      this.$refs.sig.setLineWidth()
    },
    /* 改变颜色 */
    colorChange () {
      console.log(this.color)
      this.$nextTick(function () {
        this.$refs.sig.setLineColor()
      })
    },
    /* 笔锋测试 */
    setPenShape () {
      this.$refs.sig.setPenShape()
    }
  }
}
</script>

<style scoped>
.container {
  height: 100%;
  width: 100%;
}
.selectModule {
  width: 100px;
}
.menu-item {
  padding: 0 5px;
}
.width-input {
  max-width: 80px;
}
</style>
