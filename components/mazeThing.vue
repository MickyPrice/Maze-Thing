<template>
  <div class="min-h-screen flex-col flex items-center justify-center">
    
    <!-- <p>Current Record: {{ record }}</p> -->
    <table class="mx-auto text-center text-xs aspect-square bg-white border-2 transition" :class="[ paused ? 'border-red-500' : 'border-yellow-500' ]">
      <tbody class="">
        <tr v-for="y in gridSize" :key="`row-${y-1}`" class="">
          <td v-for="x in gridSize" :key="`col-${x-1}`" class="w-8 h-8 transition rounded-full" :style="{ 'backgroundColor': previousMovesArray.includes(`${x-1},${y-1}`) ? colours[Math.floor(Math.random()*previousMovesArray.length-1) + 1] : null }" :class="{ 'bg-black' : previousMovesArray.includes(`${x-1},${y-1}`) }">
          </td>
        </tr>
      </tbody>
    </table>

    <!-- <div class="">
      <div v-for="(palette, i) in colourPalettes" :key="i" class="flex items-center w-full">
        <div v-for="(colour, j) in palette" :key="j" :style="{ backgroundColor: colour }" class="w-[10%] aspect-square" />
      </div>
    </div> -->

  </div>
</template>

<script>
export default {
  data() {
    return {
      gridSize: 20,
      previousMovesArray: ["0,0"],
      colours: ['#000000'],
      colourPalettes: [],
      interval: 0,
      record: 0,
      speed: 100,
      paused: false,
    }
  },
  computed: {
    legalMoves() {
      // Return all possible moves that wont result in the player going out of bounds (in array of strings formatted "x,y")
      // The player is currently at previousMovesArray[previousMovesArray.length-1]
      // Also don't let the player overlap themselves
      const legalMoves = []
      const currentPosition = this.previousMovesArray[this.previousMovesArray.length-1]
      const currentPositionArray = currentPosition.split(',')
      const currentX = parseInt(currentPositionArray[0])
      const currentY = parseInt(currentPositionArray[1])
      const legalMovesArray = [
        [currentX+1, currentY],
        [currentX-1, currentY],
        [currentX, currentY+1],
        [currentX, currentY-1],
      ]
      legalMovesArray.forEach(move => {
        if (move[0] >= 0 && move[0] < this.gridSize && move[1] >= 0 && move[1] < this.gridSize && !this.previousMovesArray.includes(move.join(','))) {
          legalMoves.push(move.join(','))
        }
      })

      return legalMoves
    }
  },
  watch: {
    speed: {
      handler() {
        clearInterval(this.interval)
        this.interval = setInterval(this.moveRandomly, this.speed)
      },
      immediate: true
    }
  },
  beforeDestroy() {
    clearInterval(this.interval)
  },
  methods: {
    moveRandomly() {
      if (this.legalMoves.length) {
        const randomMove = this.legalMoves[Math.floor(Math.random() * this.legalMoves.length)]
        this.colours.push(this.generateRandomColour())
        this.previousMovesArray.push(randomMove)
        if (this.previousMovesArray.length > this.record) {
          this.record = this.previousMovesArray.length
        }
      } else if (!this.paused) {
        this.paused = true
        this.getDominant10Colours(this.colours)
        setTimeout(() => {
          // Set previous moves array to a random point on the grid
          this.previousMovesArray = [`${Math.floor(Math.random() * this.gridSize)},${Math.floor(Math.random() * this.gridSize)}`]
          // Set random colour
          this.colours = [this.generateRandomColour()]
          this.paused = false
        }, 500)
      }
    },
    generateRandomColour() {
      // Generates a random hex code
      const hex = Math.floor(Math.random() * 16777215).toString(16)
      return '#' + hex
    },
    getDominant10Colours(colourArray) {
      // Take an array of hex codes, get the average of each, and return the top 10 hex codes
      const colourObject = {}
      colourArray.forEach(colour => {
        if (colourObject[colour]) {
          colourObject[colour]++
        } else {
          colourObject[colour] = 1
        }
      })
      const colourObjectArray = Object.entries(colourObject)
      colourObjectArray.sort((a, b) => b[1] - a[1])
      this.colourPalettes.push(colourObjectArray.slice(0, 10).map(colour => colour[0]))
    },
    hexToRgb(colour) {
      // Convert a hex code to an rgb object
      const hex = colour.replace('#', '')
      const r = parseInt(hex.substring(0, 2), 16)
      const g = parseInt(hex.substring(2, 4), 16)
      const b = parseInt(hex.substring(4, 6), 16)
      return {
        r,
        g,
        b
      }
    }
  },
}
</script>

<style>

</style>