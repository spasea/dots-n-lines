<template>
  <div class="hello">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
  const getRandomFloat = (max, min = 0) => Math.random() * (max - min) + min

  export default {
    name: 'HelloWorld',
    data () {
      return {
        dots: [],
        crossDots: [],
        canvasHeight: 600,
        canvasWidth: 1000,
        critical: 0,
      }
    },
    computed: {
      canvas () {
        return this.$refs.canvas
      },
      ctx () {
        return this.canvas.getContext('2d')
      }
    },
    methods: {
      generateDots () {
        this.dots.length = 0

        // mouse interaction
        this.dots.push({
          x: -this.canvasWidth,
          y: -this.canvasHeight,
        })

        this.dots.push(
          ...Array(20).fill(null).map(() => ({
            x: getRandomFloat(this.canvasWidth - this.critical, this.critical),
            y: getRandomFloat(this.canvasHeight - this.critical, this.critical),
            angle: getRandomFloat(360, 1),
            force: getRandomFloat(1.8, 0.2),
          }))
        )

        const getCombinations = (items, accu = []) => {
          if (!(items.length - 1)) return accu

          let startPoint = items[0]
          for (let idx = 1; idx < items.length; idx++) {
            accu.push([
              startPoint,
              items[idx]
            ])
          }
          return getCombinations(items.slice(1), accu)
        }

        this.crossDots = getCombinations(this.dots)
      },
      update () {
        this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)

        this.dots.slice(1).forEach(dot => {
          if (dot.x >= this.canvasWidth - this.critical || dot.x < this.critical) {
            dot.angle = 180 - dot.angle
          }
          if (dot.y >= this.canvasHeight - this.critical || dot.y < this.critical) {
            dot.angle = - dot.angle
          }

          let x = dot.x + Math.cos((360 - dot.angle) * Math.PI / 180) * dot.force
          let y = dot.y + Math.sin((360 - dot.angle) * Math.PI / 180) * dot.force
          let size = getRandomFloat(2, 1)

          this.ctx.fillRect(x, y, size, size)
          dot.x = x
          dot.y = y
        })

        // lines between neighbor dots
        // for (let j = 0; j < this.dots.length; j++) {
        //   this.drawLine(this.dots[j], this.dots[j + 1 === this.dots.length ? 0 : j + 1])
        // }

        this.crossDots.forEach(item => {
          this.drawLine(...item)
        })

        requestAnimationFrame(this.update)
      },
      drawLine (prevDot, currentDot) {
        let distance = Math.sqrt( (currentDot.x - prevDot.x)**2 + (currentDot.y - prevDot.y)**2 )

        if (distance < 150) {
          this.ctx.beginPath()
          this.ctx.moveTo(prevDot.x, prevDot.y)
          this.ctx.lineTo(currentDot.x, currentDot.y)
          this.ctx.lineWidth = 15 / distance / 2
          this.ctx.stroke()
        }
      }
    },
    mounted () {
      this.canvas.height = this.canvasHeight
      this.canvas.width = this.canvasWidth

      this.generateDots()
      this.update()

      // mouse interaction
      this.canvas.addEventListener('mousemove', e => {
        let rect = e.target.getBoundingClientRect()

        this.dots[0].x = e.clientX - rect.left
        this.dots[0].y = e.clientY - rect.top
      })

      this.canvas.addEventListener('mouseout', () => {
        this.dots[0].x = -this.canvasWidth
        this.dots[0].y = -this.canvasHeight
      })
    }
  }
</script>

<style>
  canvas {
    background: #eee;
  }
</style>
