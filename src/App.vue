<template>
  <div id="app" style="height: 100vh;width: 100vw;margin:0">
    <canvas id="pixi"></canvas>
  </div>
</template>

<script>
import * as PIXI from 'pixi.js'

export default {
  name: 'App',
  components: {},
  data: function () {
    return {
      clicks: 0,
      last: 0,
      warpspeed: 0,
      randomized: false,
      lastClick: undefined,
      lastSlow: new Date()
    }
  },
  methods: {
    onClick: function () {
      if (this.warpspeed < 5) {
        if (this.warpspeed < 2) {
          this.warpspeed = this.warpspeed + 0.3
        }

        if (this.warpspeed > 2 && this.warpspeed < 5) {
          this.warpspeed = this.warpspeed + 0.5
        }
      }
      this.lastClick = new Date()
    }
  },
  mounted () {
    var canvas = document.getElementById('pixi')


    canvas.addEventListener('mousedown', function () {

    }, false)

    const app = new PIXI.Application({
      width: window.innerWidth,
      height: window.innerHeight,
      antialias: true,
      view: canvas,
    })
    app.stage.interactive = true
    app.stage.hitArea = new PIXI.Rectangle(0, 0, window.innerWidth, window.innerHeight)
    app.stage.on('pointerdown', () => {
      if (this.warpspeed < 3.5) {
        if (this.warpspeed < 2) {
          this.warpspeed = this.warpspeed + 0.3
        }

        if (this.warpspeed > 2 && this.warpspeed < 3.5) {
          this.warpspeed = this.warpspeed + 0.5
        }
      }
      this.lastClick = new Date()
    })

// Get the texture for rope.
    const starTexture = new PIXI.Texture.from(require('./assets/star.png'))

    const starAmount = 1000
    let cameraZ = 0
    const fov = 20
    const baseSpeed = 0.025
    let speed = 0

    const starStretch = 5
    const starBaseSize = 0.05


// Create the stars
    const stars = []
    for (let i = 0; i < starAmount; i++) {
      const star = {
        sprite: new PIXI.Sprite(starTexture),
        z: 0,
        x: 0,
        y: 0,
        randomized: false
      }


      star.sprite.anchor.x = 0.5
      star.sprite.anchor.y = 0.7
      randomizeStar(star, true)
      app.stage.addChild(star.sprite)
      stars.push(star)
    }

    function randomizeStar (star, initial) {
      star.z = initial ? Math.random() * 2000 : cameraZ + Math.random() * 1000 + 2000

      // Calculate star positions with radial random coordinate so no star hits the camera.
      const deg = Math.random() * Math.PI * 2
      const distance = Math.random() * 50 + 1
      star.x = Math.cos(deg) * distance
      star.y = Math.sin(deg) * distance

    }


// Listen for animate update
    app.ticker.add((delta) => {
      // Simple easing. This should be changed to proper easing function when used for real.
      if (this.lastClick) {
        if (new Date().getTime() - this.lastClick.getTime() > 100 && new Date().getTime() - this.lastSlow.getTime() > 100 && this.warpspeed > 0) {
          this.warpspeed = this.warpspeed - 0.1 < 0 ? 0 : this.warpspeed - 0.2
          this.lastSlow = new Date()
        }
      }


      speed += (this.warpspeed - speed) / 20
      cameraZ += delta * 10 * (speed + baseSpeed)
      if (!this.randomized && this.warpspeed >= 1.1) {
        this.randomized = true
        stars.forEach(s => {
          s.sprite.tint = Math.random() * 0xFFFFFF
        })
        console.log('random')
      }
      if (this.warpspeed < 1.1) {
        stars.forEach(s => {
          s.sprite.tint = 0xFFFFFF
        })

        this.randomized = false
      }

      for (let i = 0; i < starAmount; i++) {
        const star = stars[i]
        if (star.z < cameraZ) randomizeStar(star)

        // Map star 3d position to 2d with really simple projection
        const z = star.z - cameraZ
        star.sprite.x = star.x * (fov / z) * app.renderer.screen.width + app.renderer.screen.width / 2
        star.sprite.y = star.y * (fov / z) * app.renderer.screen.width + app.renderer.screen.height / 2

        // Calculate star scale & rotation.
        const dxCenter = star.sprite.x - app.renderer.screen.width / 2
        const dyCenter = star.sprite.y - app.renderer.screen.height / 2
        const distanceCenter = Math.sqrt(dxCenter * dxCenter + dyCenter * dyCenter)
        const distanceScale = Math.max(0, (2000 - z) / 2000)
        star.sprite.scale.x = distanceScale * starBaseSize
        // Star is looking towards center so that y axis is towards center.
        // Scale the star depending on how fast we are moving, what the stretchfactor is and depending on how far away it is from the center.
        star.sprite.scale.y = distanceScale * starBaseSize + distanceScale * speed * starStretch * distanceCenter / app.renderer.screen.width
        star.sprite.rotation = Math.atan2(dyCenter, dxCenter) + Math.PI / 2


      }
    })

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

* {
  margin: 0 !important;
}
</style>
