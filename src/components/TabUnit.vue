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
                @mouseover="hoverHorizontalLine"
                @mouseleave="leaveHorizontalLine" />
      </template>
      <v-line :config="configRightVertialLine" />
    </v-layer>
  </v-stage>
</template>

<script>
import Konva from 'konva'
import _ from 'lodash'

export default {
  name: 'TabUnit',
  props: {
    width: Number,
    height: Number
  },
  data() {
    return {
      padding: 100,
      strokeWidth: 10,
      selectedLine: -1
    }
  },
  computed: {
    configStage() {
      return {
        width: this.width + this.padding * 2,
        height: this.height + this.padding * 2
      }
    },
    gap() {
      return this.height / 5
    },
    origin() {
      return {
        x: this.padding,
        y: this.padding
      }
    },
    note() {
      return {
        width: this.gap,
        textSize: 24
      }
    },
    virtualLineStrokeWidth() {
      return this.gap * 3 / 4
    },
    configLeftVertialLine() {
      return {
        points: [
          this.origin.x,
          this.origin.y,
          this.origin.x,
          this.origin.y + this.height
        ],
        stroke: 'black',
        strokeWidth: this.strokeWidth
      }
    },
    configHorizontalLines() {
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
            strokeWidth: this.strokeWidth
          },
          virtualLineConfig: {
            name: (i + 1).toString(),
            points: [x1, y, x2, y],
            stroke: 'red',
            strokeWidth: this.virtualLineStrokeWidth
          }
        }
        lines.push(line)
      }
      return lines
    },
    configRightVertialLine() {
      return {
        points: [
          this.origin.x + this.width,
          this.origin.y,
          this.origin.x + this.width,
          this.origin.y + this.height
        ],
        stroke: 'black',
        strokeWidth: this.strokeWidth
      }
    }
  },
  methods: {
    hoverHorizontalLine(horizontalLine) {
      console.log('mouse over')
      const mousePos = this.getMousePos()
      const line = horizontalLine.getStage()
      this.selectedLine = parseInt(line.getAttr('name')) - 1
      const layer = line.getLayer()
      const noteLocation = this.locateNote(line, mousePos.x)
      const shadows = layer.find('.shadow')
      if (_.isEmpty(shadows)) {
        this.createShadow(layer, noteLocation.x, noteLocation.y)
      } else {
        shadows.each(node => {
          node.remove()
        })
      }
    },
    /**
     * @param line the virtual line
     * @param x the x coordinate of mouse
     */
    locateNote(line, x) {
      const noteX =
        ((x - this.padding) % this.note.width) * this.note.width + this.padding
      const noteY = line.getAttr('points')[1] - this.virtualLineStrokeWidth / 2
      return { x: noteX, y: noteY }
    },
    leaveHorizontalLine(horizontalLine) {
       console.log('mouse leave')
      const layer = horizontalLine.getStage().getLayer()
      const shadows = layer.find('.shadow')
      shadows.forEach(node => {
        node.remove()
      })
      layer.draw()
    },
    createShadow(layer, x, y) {
      const rect = new Konva.Rect({
        name: 'shadow',
        x: x,
        y: y,
        width: this.note.width,
        height: this.note.width,
        fill: 'black'
      })
      /*
      var that = this
      rect.on('mousedown', function() {
        const y = that.configHorizontalLines[that.selectedLine].config.points[1]
        const mousePos = that.getMousePos()
        const textX = that.createTextX(mousePos.x, y)
        layer.add(textX)
        layer.draw()
      })
      */
      layer.add(rect)
      layer.draw()
    },
    createTextX(x, y) {
      return new Konva.Text({
        x: x,
        y: y,
        fontSize: 24,
        text: 'X',
        fill: 'black'
      })
    },
    getMousePos() {
      return this.$refs.stage.getStage().getPointerPosition()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
