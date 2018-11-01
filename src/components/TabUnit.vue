<template>
  <v-stage :config="configStage">
    <v-layer>
      <v-line :config="configVertialLine1" />
      <template v-for="horizontalLine in configHorizontalLines">
        <v-line :key="horizontalLine.id" :config="horizontalLine.config" />
      </template>
    </v-layer>
  </v-stage>
</template>

<script>
export default {
  name: 'TabUnit',
  props: {
    width: Number,
    height: Number,
  },
  data() {
    return {

    }
  },
  computed: {
    gap() {
      return this.height / 5
    },
    origin() {
      return {
        x: 10 ,
        y: this.padding / 2
      }
    },
    padding() {
      return this.gap
    },
    configVertialLine1() {
      return {
        points: [this.origin.x, this.origin.y, this.origin.x, this.origin.y + this.height],
        stroke: 'black',
        strokeWidth: 5,
      }
    },
    configHorizontalLines() {
      var lines = []
      for (let i = 0; i < 6; i++) {
        const y = this.origin.y + this.gap * i
        const line = {
          id: i + 1,
          config: {
            points: [this.origin.x, y, this.origin.x + this.width, y],
            stroke: 'black',
            strokeWidth: 5,
          }
        }
        lines.push(line)
      }
      return lines
    },
    configStage() {
      return {
        width: this.width,
        height: this.height + this.padding
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
