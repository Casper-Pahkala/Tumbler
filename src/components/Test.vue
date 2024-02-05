<template>
    <div id="score">
        <div>
            Score: {{ score }}
        </div>

        <div>
            High Score: {{ highScore }}
        </div>

    </div>
    <div id="circles-container">
        <div
            v-for="circle in circles"
            :key="circle.index"
            :style="`background-color: ${circle.render.fillStyle}; border-radius: 50%; width: 40px; height: 40px`"
            class="circle"
        >
        </div>

    </div>
    <div class="content">
      <div id="drawhere">
        
      </div>
    </div>

    <v-dialog
        v-model="isGameOver"
        width="600px"
        persistent
    >
        <v-card
        class="pa-5"
        >
            <v-card-title
            style="text-align: center; font-size: 30px;"
            class="mb-10"
            >
                Game Over
            </v-card-title>

            <v-btn
                color="primary"
                size="large"
                @click="$emit('resetGame')"
            >
                Play Again
            </v-btn>

        </v-card>
    </v-dialog>

</template>
  
  <script>
  import {
    Engine,
    Render,
    World,
    Bodies,
    Body,
    Events,
    Composite,
    Composites,
    Constraint,
    Vertices,
    Mouse,
    MouseConstraint,
    Query,
    Common,
    Grid,
    SAT
  } from "matter-js";
  import CryptoJS from "crypto-js";
  export default {
    data: function() {
      return {
        debug: null,
        canvasProp: {
          wallWidth: 1
        },
        canvasWidth: window.innerWidth,
        canvasHeight: window.innerHeight,
        gameScale: window.innerWidth / 1920,
        canSpawnCircle: true,
        canDrop: true,
        score: 0,
        highScore: 0,
        isGameOver: false,
        circles: [
            {
                render: {
                    fillStyle: '#D32F2F'
                },
                restitution: 0.1,
                mass: 1,
                width: 20,
                index: 0,
            },
            {
                render: {
                    fillStyle: '#EC407A'
                },
                restitution: 0.1,
                mass: 2,
                width: 28,
                index: 1,
            },
            {
                render: {
                    fillStyle: '#AB47BC'
                },
                restitution: 0.1,
                mass: 3,
                width: 38,
                index: 2
            },
            {
                render: {
                    fillStyle: '#6D4C41'
                },
                restitution: 0.1,
                mass: 4,
                width: 50,
                index: 3
            },
            {
                render: {
                    fillStyle: '#303F9F'
                },
                restitution: 0.1,
                mass: 5,
                width: 65,
                index: 4
            },
            {
                render: {
                    fillStyle: '#FFB300'
                },
                restitution: 0.1,
                mass: 6,
                width: 85,
                index: 5
            },
            {
                render: {
                    fillStyle: '#43A047'
                },
                restitution: 0.1,
                mass: 7,
                width: 110,
                index: 6
            },
            {
                render: {
                    fillStyle: '#4DB6AC'
                },
                restitution: 0.1,
                mass: 8,
                width: 130,
                index: 7
            },
            {
                render: {
                    fillStyle: '#FF7043'
                },
                restitution: 0.1,
                mass: 9,
                width: 160,
                index: 8
            },
            {
                render: {
                    fillStyle: '#33691E'
                },
                restitution: 0.1,
                mass: 10,
                width: 200,
                index: 9
            }
        ],
        keysPressed: {
            ArrowLeft: false,
            ArrowRight: false
        },
        secretKey: 'asjklfgasmfnmbashjkrgasdasnfbm,nxsabashj',
        mainContainer: {
            width: null,
            height: null
        },
      };
    },
    computed: {
      myObjStyle(top, left) {
        return `position:absolute;top:${top};left:${left}`;
      }
    },
    mounted() {
        console.log(this.gameScale);
        // this.gameScale = 0.5;

        if (window.innerWidth < 1000) {
            this.gameScale *= 2;
        }
        this.circles.forEach(circle => {
            circle.width *= this.gameScale;
            circle.mass *= this.gameScale;
            circle.restitution *= this.gameScale;
        });

        // this.canvasWidth *= this.gameScale;
        // this.canvasHeight *= this.gameScale;

        this.highScore = localStorage.getItem('highScore') ? this.decryptText(localStorage.getItem('highScore')) : 0
        let width = this.canvasWidth,
          height = this.canvasHeight,
          engineOptions = {
            timing: {
              timestamp: 0.6,
              timeScale: 0.6
            },
            gravity: {
                x: 0,
                y: 1 * this.gameScale // Adjust the gravity based on your scale factor
            },
            // broadphase: {
            //     controller: Grid,
            //     detector: SAT, // Narrow-phase collision detector
            //     buckets: {
            //         // Increase these values to improve resolution
            //         x: 10,
            //         y: 10
            //     }
            // }
            constraintIterations: 10,
            positionIterations: 10,
            velocityIterations: 10
          },
          renderOptions = {
            width,
            height,
            wireframes: false,
            wireframeBackground: "transparent",
            background: "transparent",
          };
        // create an engine
        let engine = Engine.create(engineOptions),
          world = engine.world;
        
        this.world = world;
        // create a renderer
        var render = Render.create({
          element: document.querySelector("#drawhere"),
          engine,
          options: renderOptions
        });

        const canvasWidth = this.canvasWidth;
        const canvasHeight = this.canvasHeight;
        this.mainContainer.width = 600 * this.gameScale;
        this.mainContainer.height = 700 * this.gameScale;
        const wallThickness = 10 * this.gameScale;

        console.log(canvasWidth);

        const containerCenterX = canvasWidth / 2;
        const containerCenterY = canvasHeight / 2;  

        // Left wall
        const leftWall = Bodies.rectangle(
            containerCenterX - this.mainContainer.width / 2 - wallThickness / 2,
            containerCenterY,
            wallThickness,
            this.mainContainer.height,
            { isStatic: true }
        );

        // Right wall
        const rightWall = Bodies.rectangle(
            containerCenterX + this.mainContainer.width / 2 + wallThickness / 2,
            containerCenterY,
            wallThickness,
            this.mainContainer.height,
            { isStatic: true }
        );

        // Bottom wall (ground)
        const ground = Bodies.rectangle(
            containerCenterX,
            containerCenterY + this.mainContainer.height / 2 + wallThickness / 2,
            this.mainContainer.width + 2 * wallThickness,
            wallThickness,
            { 
                isStatic: true
            }
        );
        // var img = new Image();
        // img.onload = function() {
        //     // Image is loaded, now you can use it with Matter.js or any canvas operations
        // };
        // img.src = '/dashed_line.png';

        this.topWall = Bodies.rectangle(
            containerCenterX,
            containerCenterY - this.mainContainer.height / 2 - wallThickness / 2 - 20,
            this.mainContainer.width + 2 * wallThickness,
            wallThickness,
            { 
                isStatic: true,
                isSensor: true,
                render: {
                    visible: false
                }
            }
        );
        this.createHelperLine(world);
        this.createCircle(world);
        World.add(world, [leftWall, rightWall, ground, this.topWall]);

        // run the engine
        Engine.run(engine);
        // run the renderer
        Render.run(render);

        let self = this;

        document.addEventListener('keydown', function(event) {
            if (event.key === 'ArrowLeft' || event.key === 'ArrowRight') {
                self.keysPressed[event.key] = true;
                event.preventDefault();  // Prevent browser default behavior
            }
            if (event.code === 'Space' && self.canDrop) {
                self.dropCircle(world, self.currentCircle.index);
                event.preventDefault();
            }
        });

        document.addEventListener('keyup', function(event) {
            if (event.key === 'ArrowLeft' || event.key === 'ArrowRight') {
                self.keysPressed[event.key] = false;
            }
        });

        this.moveObjects();

        Events.on(engine, 'collisionStart', (event) => {
            for (let pair of event.pairs) {
                const bodyA = pair.bodyA;
                const bodyB = pair.bodyB;

                if (bodyA.id != bodyB.id && bodyA.index === bodyB.index && !bodyA.isStatic && !bodyB.isStatic) {
                    // Both bodies are circles with the same radius
                    
                    // Calculate the midpoint between the two circles
                    const midX = (bodyA.position.x + bodyB.position.x) / 2;
                    const midY = (bodyA.position.y + bodyB.position.y) / 2;

                    // Remove the two colliding circles
                    World.remove(engine.world, bodyA);
                    World.remove(engine.world, bodyB);

                    self.score += bodyA.index + 1;

                    if (self.score > self.highScore) {
                        self.highScore = self.score;
                        localStorage.setItem('highScore', self.encryptText(self.highScore.toString()));
                    }

                    let circle = self.circles[bodyA.index + 1];

                    // Create a new circle with 1.5x the width
                    const newCircle = Bodies.circle(midX, midY, circle.width, circle);

                    World.add(world, newCircle);
                }

                if ((bodyA === self.droppingCircle || bodyB === self.droppingCircle) && !bodyA.isSensor && !bodyB.isSensor && bodyA !== self.topWall && bodyB !== self.topWall) {
                    // console.log('top', self.topWall);
                    // console.log(bodyA, bodyB);
                    this.circleDropping = false;
                    setTimeout(() => {
                        self.startTurn(world);
                    }, 10);
                }
            }
        });

        Events.on(engine, 'collisionActive', (event) => {
            for (let pair of event.pairs) {
                const bodyA = pair.bodyA;
                const bodyB = pair.bodyB;

                if (!self.circleDropping && (bodyA === self.topWall || bodyB === self.topWall)) {
                    self.gameOver(engine);
                }
            }
        });

        this.mainCanvas = document.querySelector("#drawhere").children[0];


        this.mainCanvas.addEventListener('mousedown', this.handleStart);
        this.mainCanvas.addEventListener('mousemove', this.handleMove);
        this.mainCanvas.addEventListener('mouseup', this.handleEnd);

        this.mainCanvas.addEventListener('touchstart', this.handleStart);
        this.mainCanvas.addEventListener('touchmove', this.handleMove);
        this.mainCanvas.addEventListener('touchend', this.handleEnd);
      },
      methods: {
        gameOver(engine) {
            Composite.allBodies(engine.world).forEach(function(body) {
                Body.setVelocity(body, { x: 0, y: 0 });
                Body.setAngularVelocity(body, 0);
                Body.setStatic(body, true);
            });
            this.isGameOver = true;
        },
        moveObjects() {
            if (this.isGameOver) return;

            let self = this;
            const moveAmount = 3 * this.gameScale;

            if (this.keysPressed.ArrowLeft) {
                let newPos = self.helperLine.position.x - moveAmount;

                let offset = (this.canvasWidth - this.mainContainer.width) / 2;
                if (newPos <= (offset + this.currentCircle.width)) {
                    newPos = offset + this.currentCircle.width;
                }
                Body.setPosition(self.helperLine, { x: newPos, y: self.helperLine.position.y });
                Body.setPosition(self.currentCircle, { x: newPos, y: self.currentCircle.position.y });
            }
            if (this.keysPressed.ArrowRight) {
                let newPos = self.helperLine.position.x + moveAmount;
                let offset = (this.canvasWidth + this.mainContainer.width) / 2;
                if (newPos >= (offset - this.currentCircle.width)) {
                    newPos = offset - this.currentCircle.width;
                }
                Body.setPosition(self.helperLine, { x: newPos, y: self.helperLine.position.y });
                Body.setPosition(self.currentCircle, { x: newPos, y: self.currentCircle.position.y });
            }
            requestAnimationFrame(this.moveObjects);
        },
        createHelperLine(world) {
            const lineWidth = 3 * this.gameScale;
            const lineHeight = this.mainContainer.height - (20 * this.gameScale);
            const initialX = (this.canvasWidth / 2);  // Assuming a canvas width of 800
            const initialY = ((this.canvasHeight / 2) + (20 * this.gameScale));

            this.helperLine = Bodies.rectangle(initialX, initialY, lineWidth, lineHeight, {
                isStatic: true,
                isSensor: true,  // This ensures it doesn't participate in collisions
                render: {
                    fillStyle: '#fff'
                }
            });

            World.add(world, this.helperLine);
        },
        createCircle(world) {
            if (this.isGameOver) return;
            const circleX = this.helperLine.position.x;
            const circleY = (this.canvasHeight / 2) - (this.mainContainer.height / 2) - (20 * this.gameScale);

            if (!this.nextCirle) {
                const randomInteger = this.randomNumber();
                this.nextCirle = JSON.parse(JSON.stringify(this.circles[randomInteger]))
            }

            if (this.nextDisplayCircle) {
                World.remove(world, this.nextDisplayCircle);
            }

            let _circle = this.nextCirle;
            _circle.isStatic = true;
            _circle.isSensor = true;
            this.currentCircle = Bodies.circle(circleX, circleY, _circle.width, _circle);
            World.add(world, [this.currentCircle]);

            this.nextCirle = JSON.parse(JSON.stringify(this.circles[this.randomNumber()]));

            let next = this.nextCirle;
            next.isStatic = true;
            next.isSensor = true;

            let nextDisplayCircleX = this.canvasWidth / 2;
            nextDisplayCircleX += this.mainContainer.width / 2;
            nextDisplayCircleX += 20 * this.gameScale + this.nextCirle.width;

            this.nextDisplayCircle = Bodies.circle(nextDisplayCircleX, circleY, next.width, next);
            World.add(world, [this.nextDisplayCircle]);
        },
        dropCircle(world, index) {
            if (this.isGameOver) return;
            this.circleDropping = true;
            const circleX = this.helperLine.position.x;
            const circleY = (this.canvasHeight / 2) - (this.mainContainer.height / 2) - (20 * this.gameScale);

            let _circle = JSON.parse(JSON.stringify(this.circles[index]));
            _circle.mass = _circle.width * 0.05;
            this.droppingCircle = Bodies.circle(circleX, circleY, _circle.width, _circle);
            World.add(world, [this.droppingCircle]);

            World.remove(world, this.currentCircle);

            // this.createCircle(world);

            this.canDrop = false;
            // this.startTurn(world);
        },

        startTurn(world) {
            if (this.isGameOver) return;
            if (!this.droppingCircle) return;

            this.droppingCircle = null;

            const circleY = (this.canvasHeight / 2) - (this.mainContainer.height / 2) - (20 * this.gameScale);            
            
            let helperLinePos = this.helperLine.position.x;

            let offset = (this.canvasWidth - this.mainContainer.width) / 2;
            if (helperLinePos <= (offset + this.nextCirle.width)) {
                helperLinePos = offset + this.nextCirle.width;
            }
            offset = (this.canvasWidth + this.mainContainer.width) / 2;
            if (helperLinePos >= (offset - this.nextCirle.width)) {
                helperLinePos = offset - this.nextCirle.width;
            }
            Body.setPosition(this.helperLine, { x: helperLinePos, y: this.helperLine.position.y });
            
            
            const randomInteger = this.randomNumber();
            

            let _circle = this.nextCirle;
            _circle.isStatic = true;
            _circle.isSensor = true;
            this.currentCircle = Bodies.circle(helperLinePos, circleY, _circle.width, _circle);
            World.add(world, [this.currentCircle]);

            World.remove(world, this.nextDisplayCircle);

            this.nextCirle = JSON.parse(JSON.stringify(this.circles[randomInteger]))

            this.canDrop = true;
            let next = this.nextCirle;
            next.isStatic = true;
            next.isSensor = true;

            let nextDisplayCircleX = this.canvasWidth / 2;
            nextDisplayCircleX += this.mainContainer.width / 2;
            nextDisplayCircleX += 20 * this.gameScale + this.nextCirle.width;

            this.nextDisplayCircle = Bodies.circle(nextDisplayCircleX, circleY, next.width, next);
            World.add(world, [this.nextDisplayCircle]);


        },

        randomNumber() {
            const rand = Math.random();
            if (rand < 0.15) {
                return 4;
            } else if (rand < 0.15 + 0.18) { // 0.33
                return 3;
            } else if (rand < 0.15 + 0.18 + 0.20) { // 0.53
                return 2;
            } else if (rand < 0.15 + 0.18 + 0.20 + 0.22) { // 0.75
                return 1;
            } else {
                return 0;
            }
        },
        encryptText(text) {
            return CryptoJS.AES.encrypt(text, this.secretKey).toString();
        },
        decryptText(text) {
            const bytes = CryptoJS.AES.decrypt(text, this.secretKey);
            return bytes.toString(CryptoJS.enc.Utf8);
        },
        handleStart(event){
            this.startedDrag = true;
            this.handleMove(event);
        },
        handleMove(event) {
            if (!this.startedDrag) return;
            console.log(event);

            let newPos = event.x ?? event.touches[0].clientX;

            let offset = (this.canvasWidth - this.mainContainer.width) / 2;
            if (newPos <= (offset + this.currentCircle.width)) {
                newPos = offset + this.currentCircle.width;
            }

            offset = (this.canvasWidth + this.mainContainer.width) / 2;
            if (newPos >= (offset - this.currentCircle.width)) {
                newPos = offset - this.currentCircle.width;
            }
            Body.setPosition(this.helperLine, { x: newPos, y: this.helperLine.position.y });
            Body.setPosition(this.currentCircle, { x: newPos, y: this.currentCircle.position.y });
        },
        handleEnd(event) {
            if (this.canDrop) {
                this.dropCircle(this.world, this.currentCircle.index);
                this.startedDrag = false;
            }
            console.log(event);
        }
      }
      
  };
  </script>
  
  
  <style scoped>

  #circles-container {
    position: absolute;
    left: 50px;
    top: 150px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 30px;
    justify-content: center;

  }

  #score {
    position: absolute;
    top: 50px;
    left: 50px;
    font-size: 24px;
    color: #fff;
  }

  .circle {
    position: relative;
  }

  .circle:not(:last-child)::after {
    content: "⬇️";
    position: absolute;
    bottom: -25px;
    left: 10px;
    color: #fff;
    /* margin-top: 80px; */
  }

  @media (max-width: 1199px) {
    #score {
        /* display: none; */
    }

    #circles-container {
        display: none;
    }
    
  }
  </style>