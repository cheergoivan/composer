<template>
  <v-stage :config="configStage"
           ref="stage">
    <v-layer>
      <v-line :config="configLeftVertialLine" />
      <template v-for="horizontalLine in configHorizontalLines">
        <v-line :key="'v' + horizontalLine.id"
                :config="horizontalLine.config" />
        <v-line :key="horizontalLine.id"
                :config="horizontalLine.virtualLineConfig"
                @mousemove="hoverHorizontalLine"
                @mouseleave="leaveHorizontalLine" />
      </template>
      <v-line :config="configRightVertialLine" />
    </v-layer>
  </v-stage>
</template>

<script>
import Konva from 'konva'

export default {
  name: 'TabUnit',
  props: {
    width: Number,
    height: Number,
  },
  data () {
    return {
      padding: 100,
      strokeWidth: 5,
      selectedLine: -1
    }
  },
  computed: {
    configStage () {
      return {
        width: this.width + this.padding * 2,
        height: this.height + this.padding * 2
      }
    },
    gap () {
      return this.height / 5
    },
    origin () {
      return {
        x: this.padding,
        y: this.padding
      }
    },
    configLeftVertialLine () {
      return {
        points: [this.origin.x, this.origin.y, this.origin.x, this.origin.y + this.height],
        stroke: 'black',
        strokeWidth: this.strokeWidth,
      }
    },
    configHorizontalLines () {
      var lines = []
      for (let i = 0; i < 6; i++) {
        const x1 = this.origin.x - this.strokeWidth / 2
        const x2 = x1 + this.width + this.strokeWidth
        const y = this.origin.y + this.gap * i - this.strokeWidth / 2
        const line = {
          id: i + 1,
          config: {
            name: (i + 1).toString(),
            points: [x1, y, x2, y],
            stroke: 'black',
            strokeWidth: this.strokeWidth,
          },
          virtualLineConfig: {
            name: (i + 1).toString(),
            points: [x1, y, x2, y],
            stroke: 'transparent',
            strokeWidth: this.gap / 2
          }
        }
        lines.push(line)
      }
      return lines
    },
    configRightVertialLine () {
      return {
        points: [this.origin.x + this.width, this.origin.y, this.origin.x + this.width, this.origin.y + this.height],
        stroke: 'black',
        strokeWidth: this.strokeWidth
      }
    }
  },
  methods: {
    hoverHorizontalLine (horizontalLine) {
      const mousePos = this.getMousePos()
      const line = horizontalLine.getStage()
      this.selectedLine = parseInt(line.getAttr('name')) - 1
      const layer = line.getLayer()
      layer.find('.shadow').each((node) => {
        node.remove()
      })
      const shadowRect = this.createShadow(mousePos.x, mousePos.y)
      layer.add(shadowRect)
      layer.draw()
    },
    leaveHorizontalLine (horizontalLine) {
      const layer = horizontalLine.getStage().getLayer()
      layer.find('.shadow').each((node) => {
        node.remove()
      })
      layer.draw()
    },
    createShadow (layer, x, y) {
      const length = this.gap
      const rect = new Konva.Rect({
        name: 'shadow',
        x: x - length / 2,
        y: y - length / 2,
        width: length,
        height: length,
        fill: 'red'
      })
      var that = this
      rect.on('mousedown', function () {
        const y = that.configHorizontalLines[that.selectedLine].config.points[1]
        const mousePos = that.getMousePos()
        const textX = that.createTextX(mousePos.x, y)
        layer.add(textX)
        layer.draw()
      })
      layer.add(rect)
      layer.draw()
    },
    createTextX (x, y) {
      return new Konva.Text({
        x: x,
        y: y,
        fontSize: 24,
        text: 'X',
        fill: 'black'
      })
    },
    getMousePos () {
      return this.$refs.stage.getStage().getPointerPosition()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
