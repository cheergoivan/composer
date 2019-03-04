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
      this.removePromptNote(layer)
      if (_.findIndex(this.occupied, noteLocation) < 0) {
        this.createPromptNote(layer, noteLocation)
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
    createNoteContainer (layer, noteLocation, name) {
      const noteContainer = new Konva.Rect({
        name: name,
        x: noteLocation.x,
        y: noteLocation.y,
        width: this.note.width,
        height: this.note.width,
        fill: 'red'
      })
      layer.add(noteContainer)
      return noteContainer
    },
    createPromptNoteContainer (layer, noteLocation) {
      this.createNoteContainer(layer, noteLocation, 'promptNoteContainer')

    },
    createPersistNoteContainer (layer, noteLocation) {
      this.createNoteContainer(layer, noteLocation, 'noteContainer')
    },
    createNoteText (layer, noteLocation, name, text) {
      const noteText = new Konva.Text({
        name: name,
        x: noteLocation.x,
        y: noteLocation.y,
        width: this.note.width,
        height: this.note.width,
        align: 'center',
        verticalAlign: 'middle',
        fontSize: this.note.textSize,
        text: text,
        fontStyle: 'bold',
        fill: 'black'
      })
      layer.add(noteText)
      return noteText
    },
    createPromptNoteText (layer, noteLocation) {
      const noteText = this.createNoteText(layer, noteLocation, 'promptNote', 'X')
      noteText.on('click', () => {
        this.removePromptNote(layer)
        this.createPersistNote(layer, noteLocation, 'X')
      })
      noteText.on('mouseout', () => {
        this.removePromptNote(layer)
      })
    },
    createPersistNoteText (layer, noteLocation, text) {
      const noteText = this.createNoteText(layer, noteLocation, 'note', text)
      var clickTimer = null
      noteText.on('dblclick', () => {
        //clearTimeout(clickTimer)
        const textarea = this.createTextarea(noteText)
        const that = this
        textarea.addEventListener('keydown', function (e) {
          if (e.keyCode === 13) {
            //noteText.text(textarea.value.toUpperCase())
            //layer.draw()
            document.body.removeChild(textarea)
            if (textarea.value) {
              that.createPersistNote(layer, noteLocation, textarea.value.toUpperCase())
            }
          }
        })
      })
      noteText.on('click', () => {
        clearTimeout(clickTimer)
        clickTimer = setTimeout(() => {
          noteText.remove()
          layer.draw()
          _.remove(this.occupied, n => {
            return _.eq(n, noteLocation)
          })
        }, 200)
      })
    },
    createPersistNote (layer, noteLocation, text) {
      //this.createPersistNoteContainer(layer, noteLocation)
      this.createPersistNoteText(layer, noteLocation, text)
      layer.draw()
      this.occupied.push(noteLocation)
    },
    createPromptNote (layer, noteLocation) {
      //this.createPromptNoteContainer(layer, noteLocation)
      this.createPromptNoteText(layer, noteLocation)
      layer.draw()
    },
    removePromptNote (layer) {
      this.removeNote(layer, 'promptNote')
    },
    removePersistNote (layer) {
      this.removeNote(layer, 'note')
    },
    removeNote (layer, name) {
      const noteContainers = layer.find('.' + name + 'Container')
      noteContainers.each(node => {
        node.remove()
      })
      const notes = layer.find('.' + name)
      notes.each(node => {
        node.remove()
      })
      layer.draw()
    },
    createTextarea: function (note) {
      var noteAbsPosition = note.getAbsolutePosition()
      var stageBox = this.getStage().getContainer().getBoundingClientRect()
      var textareaPosition = {
        x: noteAbsPosition.x + stageBox.left,
        y: noteAbsPosition.y + stageBox.top
      }
      var textarea = document.createElement('input')
      document.body.appendChild(textarea)
      //textarea.value = note.text()
      textarea.style.position = 'absolute'
      textarea.style.top = textareaPosition.y + 'px'
      textarea.style.left = textareaPosition.x + 'px'
      const border = 2, padding = 0, margin = 0
      textarea.style.width = note.width() - 2 * (border + padding + margin) + 'px'
      textarea.style.height = note.height() - 2 * (border + padding + margin) + 'px'
      textarea.style.fontFamily = 'sans-serif'
      textarea.style.fontWeight = 600
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
