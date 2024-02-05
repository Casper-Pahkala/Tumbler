<template>
    <div tabindex="0" ref="gameContainer" class="game-container"></div>
  </template>
  
  <script>
import { Engine, Render, Bodies, World, Runner, Composite, Body } from 'matter-js';

  
  export default {
    data() {
      return {
        // engine: null,
        circles: [],
        // helperLine: null,
        canvasWidth: 800,
        canvasHeight: 800,
        // other game state data
      };
    },
    mounted() {
        this.engine = Engine.create();

        // create a renderer
        this.renderer = Render.create({
            element: this.$refs.gameContainer,
            engine: this.engine,
            options: {
              width: this.canvasWidth,
              height: this.canvasHeight,
              // wireframes: false
            }
        });

        // create two boxes and a ground
        // var boxA = Bodies.rectangle(400, 200, 80, 80);
        // var boxB = Bodies.rectangle(450, 50, 80, 80);
        // // var ground = Bodies.rectangle(400, 610, 810, 60, { isStatic: true });

        // // add all of the bodies to the world
        // Composite.add(this.engine.world, [boxA, boxB]);

        // for (let i = 0; i < 10; i++) {
        //     let circle = Bodies.circle(400, 10 * 5, 50, {
        //       render: {
        //           fillStyle: '#F00',
        //           strokeStyle: '#FFF',
        //           lineWidth: 3
        //       }
        //   });
        //   World.add(this.engine.world, circle);
        // }
        this.createContainer();
        this.createHelperLine();
        Engine.run(this.engine);
        // run the renderer
        Render.run(this.renderer);

        let self = this;
        document.addEventListener('keydown', function(event) {
            if (event.code === 'Space') {
                // Spawn a circle when space key is pressed
                const circle = Bodies.circle(400, 300, 20, {
                    render: {
                        fillStyle: 'red'
                    },
                    // restitution: 0
                });
                World.add(self.engine.world, [circle]);

                console.log("Circle Velocity:", circle.velocity);

            }
        });
    },
    methods: {
      dropCircle() {
        const circleX = this.helperLine.position.x;
        const circleY = this.helperLine.position.y - (this.helperLine.bounds.max.y - this.helperLine.bounds.min.y) / 2 - 50;
        const circle = Bodies.circle(circleX, circleY, 50, {
            render: {
              fillStyle: '#F00',
              strokeStyle: '#FFF',
              lineWidth: 3
            }
        });
        World.add(this.engine.world, circle);
      },
      combineCircles(circle1, circle2) {
        // Logic to combine two circles
        // Remove old circles and add a bigger one
      },

      createContainer() {
        const canvasWidth = this.canvasWidth;
        const canvasHeight = this.canvasHeight;
        const containerWidth = 400;
        const containerHeight = 800;
        const wallThickness = 10;

        const containerCenterX = canvasWidth / 2;
        const containerCenterY = canvasHeight / 2;  

        // Left wall
        const leftWall = Bodies.rectangle(
            containerCenterX - containerWidth / 2 - wallThickness / 2,
            containerCenterY,
            wallThickness,
            containerHeight,
            { isStatic: true }
        );

        // Right wall
        const rightWall = Bodies.rectangle(
            containerCenterX + containerWidth / 2 + wallThickness / 2,
            containerCenterY,
            wallThickness,
            containerHeight,
            { isStatic: true }
        );

        // Bottom wall (ground)
        const ground = Bodies.rectangle(
            containerCenterX,
            containerCenterY + containerHeight / 2 + wallThickness / 2,
            containerWidth + 2 * wallThickness,
            wallThickness,
            { isStatic: true }
        );


        World.add(this.engine.world, [leftWall, rightWall, ground]);

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
      },
      handleKey(event) {
        console.log(event);
        const moveAmount = 10;

        if (event.key === 'ArrowLeft') {
            Body.setPosition(this.helperLine, { x: this.helperLine.position.x - moveAmount, y: this.helperLine.position.y });
        } else if (event.key === 'ArrowRight') {
            Body.setPosition(this.helperLine, { x: this.helperLine.position.x + moveAmount, y: this.helperLine.position.y });
        } else if (event.code === 'Space') {
          this.dropCircle();
        }
        // Engine.update(this.engine);
      }
      // other game methods
    },
    beforeDestroy() {
      // Cleanup, stop the physics engine, etc.
    }
  };
  </script>
  
<style scoped>
  .game-container {
    /* Style your game container */
    /* border: 1px solid #fff;
    border-top: none; */
    /* width: 400px;
    height: 600px; */
    width: 100vw;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

    .game-container canvas {
        /* border: 1px solid #fff; */
        width: 800px;
        height: 600px;
    }
</style>
  