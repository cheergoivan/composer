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
                @mousemove="hoverHorizontalLine" />
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
  data () {
    return {
      padding: 100,
      strokeWidth: 5,
      occupied: []
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
    note () {
      return {
        width: this.gap,
        textSize: this.gap * 3 / 4
      }
    },
    virtualLineStrokeWidth () {
      return this.gap * 3 / 4
    },
    configLeftVertialLine () {
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
            strokeWidth: this.strokeWidth
          },
          virtualLineConfig: {
            name: (i + 1).toString(),
            points: [x1, y, x2, y],
            stroke: 'transparent',
            strokeWidth: this.virtualLineStrokeWidth
          }
        }
        lines.push(line)
      }
      return lines
    },
    configRightVertialLine () {
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
    hoverHorizontalLine (horizontalLine) {
      const mousePos = this.getMousePos()
      const line = horizontalLine.getStage()
      const layer = line.getLayer()
      const noteLocation = this.locateNote(line, mousePos.x)
      const shadows = layer.find('.noteContainer')
      shadows.each(node => {
        node.remove()
        layer.draw()
      })
      if (_.findIndex(this.occupied, this.locateNoteText(noteLocation)) < 0) {
        this.createNote(layer, noteLocation)
      }
    },
    /**
     * @param line the virtual line
     * @param x the x coordinate of mouse
     */
    locateNote (line, x) {
      const noteX =
        (Math.floor((x - this.padding) / this.note.width)) * this.note.width + this.padding
      const noteY = line.getAttr('points')[1] - this.note.width / 2
      return { x: noteX, y: noteY }
    },
    createNote (layer, noteLocation) {
      const note = new Konva.Rect({
        name: 'noteContainer',
        x: noteLocation.x,
        y: noteLocation.y,
        width: this.note.width,
        height: this.note.width,
        fill: 'red'
      })
      note.on('mousedown', () => {
        this.createNoteText(layer, note)
      })
      layer.add(note)
      layer.draw()
    },
    locateNoteText (noteLocation) {
      return { x: noteLocation.x + this.note.width / 2 - this.note.textSize / 3, y: noteLocation.y + this.note.width / 2 - this.note.textSize / 2 }
    },
    createNoteText (layer, note) {
      const noteLocation = {
        x: note.getAttr('x'),
        y: note.getAttr('y')
      }
      const noteTextLocation = this.locateNoteText(noteLocation)
      this.occupied.push(noteTextLocation)
      var textNode = new Konva.Text({
        name: 'note',
        x: noteTextLocation.x,
        y: noteTextLocation.y,
        fontSize: this.note.textSize,
        text: 'X',
        fontStyle: 'bold',
        fill: 'black'
      })
      layer.add(textNode)
      layer.draw()
      textNode.on('click', () => {
        const textarea = this.createTextarea(note, textNode)
        textarea.addEventListener('keydown', function (e) {
          if (e.keyCode === 13) {
            textNode.text(textarea.value.toUpperCase())
            note.remove()
            layer.draw()
            document.body.removeChild(textarea)
          }
        })
      })
    },
    createTextarea: function (note, textNode) {
      var noteAbsPosition = note.getAbsolutePosition()
      var stageBox = this.getStage().getContainer().getBoundingClientRect()
      var textareaPosition = {
        x: noteAbsPosition.x + stageBox.left,
        y: noteAbsPosition.y + stageBox.top
      }
      var textarea = document.createElement('input')
      document.body.appendChild(textarea)
      textarea.value = textNode.text()
      textarea.style.position = 'absolute'
      textarea.style.top = textareaPosition.y + 'px'
      textarea.style.left = textareaPosition.x + 'px'
      const border = 2, padding = 0, margin = 0
      textarea.style.width = note.width() - 2 * (border + padding + margin) + 'px'
      textarea.style.height = note.width() - 2 + 'px'
      textarea.style.fontSize = textNode.width() + 'px'
      textarea.style.fontSize = this.note.textSize + 'px'
      textarea.style.borderWidth = border + 'px'
      textarea.style.padding = padding + 'px'
      textarea.style.margin = margin + 'px'
      textarea.style.textAlign = 'center'
      textarea.style.textTransform = 'uppercase'
      textarea.focus()
      return textarea
    },
    getStage () {
      return this.$refs.stage.getStage()
    },
    getMousePos () {
      return this.getStage().getPointerPosition()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
