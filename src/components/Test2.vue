<template>
    <div @keydown="handleKey" tabindex="0" ref="gameContainer" class="game-container"></div>
  </template>
  
  <script>
import { Engine, Render, Bodies, World, Runner, Composite, Body } from 'matter-js';

  
  export default {
    data() {
      return {
        engine: null,
        circles: [],
        helperLine: null,
        canvasWidth: window.innerWidth,
        canvasHeight: window.innerHeight - 55,
        canvasProp: {
          wallWidth: 1
        }
        // other game state data
      };
    },
    mounted() {
        this.engine = Engine.create({
            timing: {
              timestamp: 0.5,
              timeScale: 0.5
            }
        });
        let world = this.engine.world;
        // create a renderer
        let render = Render.create({
            element: this.$refs.gameContainer,
            engine: this.engine,
            options: {
              width: this.canvasWidth,
              height:this.canvasHeight,
              // wireframes: false
            }
        });

        this.createContainer();
        Engine.run(this.engine);
        // run the renderer
        Render.run(render);

        document.addEventListener('keydown', function(event) {
            if (event.code === 'Space') {
                // Spawn a circle when space key is pressed
                const circle = Bodies.circle(400, 300, 20, {
                    render: {
                        fillStyle: 'red'
                    },
                    // restitution: 0
                });
                World.add(world, [circle]);

                console.log("Circle Velocity:", circle.velocity);

            }
        });
    },
    methods: {
      dropCircle() {
        const circle = Bodies.circle(400, 300, 20, {
            render: {
                fillStyle: 'red'
            },
            // restitution: 0
        });
        World.add(this.engine.world, [circle]);

        console.log("Circle Velocity:", circle.velocity);
      },
      combineCircles(circle1, circle2) {
        // Logic to combine two circles
        // Remove old circles and add a bigger one
      },

      createContainer() {
        let width = window.innerWidth,
          height = window.innerHeight - 55;
        let leftWall = Bodies.rectangle(
            this.canvasProp.wallWidth,
            height / 2,
            this.canvasProp.wallWidth,
            height,
            { isStatic: true }
          ),
          rightWall = Bodies.rectangle(
            width - this.canvasProp.wallWidth,
            height / 2,
            this.canvasProp.wallWidth,
            height,
            { isStatic: true }
          ),
          topWall = Bodies.rectangle(
            width / 2,
            this.canvasProp.wallWidth,
            width,
            this.canvasProp.wallWidth,
            { isStatic: true }
          ),
          bottomWall = Bodies.rectangle(
            width / 2,
            height - this.canvasProp.wallWidth,
            width,
            this.canvasProp.wallWidth,
            { 
                isStatic: true,
                // restitution: 0
            }
          );


        World.add(this.engine.world, [leftWall, rightWall, topWall, bottomWall]);

      },
      createHelperLine() {
        const lineWidth = 3;
        const lineHeight = this.canvasHeight - 200;
        const initialX = this.canvasWidth / 2;  // Assuming a canvas width of 800
        const initialY = this.canvasHeight / 2;

        this.helperLine = Bodies.rectangle(initialX, initialY, lineWidth, lineHeight, {
            isStatic: true,
            isSensor: true,  // This ensures it doesn't participate in collisions
            render: {
                fillStyle: '#fff'
            }
        });

        World.add(this.engine.world, this.helperLine);
      }
    }
  };
  </script>
  
<style scoped>
</style>
  