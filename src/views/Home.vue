<template>
  <div class="home">
    <canvas ref="can" width="200" height="200"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import { fabric } from 'fabric'

export default {
  name: 'Home',
  components: {},
  mounted () {
    const ref = this.$refs.can
    const canvas = new fabric.Canvas(ref)
    const rect = new fabric.Rect({
      fill: 'red',
      width: 20,
      height: 20
    })
    canvas.add(rect)
  }
}
</script>
