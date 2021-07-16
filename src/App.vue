<template>
    <div>
        <iv-visualisation :title="projectName">
            <template #hotspots>
                <!-- <iv-pane position="left" format="full">
                    <iv-sidebar-content :showPagination="true">
                        <iv-sidebar-section title="Instructions" icon="book-open">
                            Test
                        </iv-sidebar-section>
                    </iv-sidebar-content>
                </iv-pane> -->

                <iv-fixed-hotspot position="right">
                    <div style="width: 100%;">
                        <iv-slider id="mass1Slider" name="Mass (kg)" :min="0.1" :max="10" :step="0.1" :tick_step="9.9" :init_val="0.1" @sliderChanged="changeMass1"/>
                        <iv-slider id="velocity1Slider" name="Velocity X (m/s)" :min="0" :max="10" :step="0.1" :tick_step="10" :init_val="0" @sliderChanged="changeVelX"/>
                        <iv-slider id="mass2Slider" name="Mass (kg)" :min="0.1" :max="10" :step="0.1" :tick_step="9.9" :init_val="0.1" @sliderChanged="changeMass2"/>
                        <iv-slider id="angleSlider" name="Collision angle" :min="-90" :max="90" :step="5" :tick_step="180" :init_val="50" @sliderChanged="changeAngle"/>
                    </div>
                </iv-fixed-hotspot>
            </template>

            <!-- Game -->
            <div class="center" id="canvasWrapper">
                
            </div>
        </iv-visualisation>
    </div>
</template>
<script>
// import {name} from '../package.json';
import $ from 'jquery'; // eslint-disable-line no-unused-vars
import Phaser from "phaser"; // eslint-disable-line no-unused-vars
 
export default {
    name:"App",
    data() {
        return {
            projectName: "Collisions",
            ball1Mass: 0.1,
            ball2Mass: 0.1,
            velocityX: 0,
            collisionAngle: 50,
        };
    },
    methods: {
        changeMass1(e) {
            this.ball1Mass = e;
            console.log(`Ball 1 mass: ${this.ball1Mass}`);
        },
        changeMass2(e) {
            this.ball2Mass = e;
            console.log(`Ball 2 mass: ${this.ball2Mass}`);
        },
        changeVelX(e) {
            this.velocityX = e;
            console.log(`Velocity X: ${this.velocityX}`);
        },
        changeAngle(e) {
            this.collisionAngle = e;
            console.log(`Collision angle: ${this.collisionAngle}`);
        }
    },
    mounted() {
        console.log("Start");

        let vm = this;
        
        function Vector(x, y) {
            this.x = x;
            this.y = y;
            this.getArg = function () {
                return Math.atan2(this.y, this.x);
            };
            this.getMag = function () {
                return Math.sqrt(Math.pow(this.x, 2) + Math.pow(this.y, 2));
            };
            this.toString = function () {
                return "(" + this.x.toString() + "," + this.y.toString() + ")";
            };
        }

        function vCal(input1, action, input2) {
            //Vector Calculations
            switch (action) {
                case "-":
                    return new Vector(input1.x - input2.x, input1.y - input2.y);
                case "+":
                    return new Vector(input1.x + input2.x, input1.y + input2.y);
                case "*":
                    return new Vector(input2 * input1.x, input2 * input1.y);
                case "rotate":
                    return new Vector(input1.x * Math.cos(input2) - input1.y * Math.sin(input2), input1.x * Math.sin(input2) + input1.y * Math.cos(input2));
                default:
                    console.log("undefined operation, " + action);

            }
        }

        function doPhysics(ball1,ball2,scatterAngle){
            let reducedMass =( ball1.mass*ball2.mass) / (ball2.mass + ball1.mass);
            let pStar = vCal(vCal(ball2.initV, '-', ball1.initV), '*',reducedMass);
            let q1star = vCal(pStar, 'rotate', scatterAngle);
            let q2star = vCal(pStar, 'rotate', scatterAngle - Math.PI);
            return [vCal(q1star, '*', 1 / ball1.mass), vCal(q2star, '*', 1 / ball2.mass)];
        }

        /*
        ++++++++++++++++++++++++++++++++++++
        Pre-defined classes
        ++++++++++++++++++++++++++++++++++++
        */

        // class ToolTip{
        //     constructor(position,title,text){
        //         this.position = position;

        //         this.showRect = sceneGame.add.rectangle(position.x - showRange/2,position.y - showRange/2,showRange,showRange);
        //         let roundedRectG =  sceneGame.add.graphics(0,0);
        //         // roundedRectG.beginFill(0x003E74);
        //         roundedRectG.fillRoundedRect(0,0,200,(fontSize + 7)*4,5);
        //         this.sprite = uiGroup.create(position.x,position.y,roundedRectG.generateTexture());
        //         this.title = sceneGame.add.text(position.x + 5,position.y,title,{font:"20px Fira Sans",fill:"#ffffff"});
        //         uiGroup.add(this.title);
        //         this.text = sceneGame.add.text(position.x + 5,position.y + fontSize+10,text,{font:"12px Fira Sans",fill:"#ffffff"});
        //         uiGroup.add(this.text);
        //         roundedRectG.destroy();
        //     }
        //     update(){
        //         if(game.input.mousePointer.x < this.showRect.x ||
        //         game.input.mousePointer.x > this.showRect.x + this.showRect.width ||
        //             game.input.mousePointer.y < this.showRect.y ||
        //             game.input.mousePointer.y > this.showRect.y + this.showRect.height) {
        //                 this.sprite.visible = false;
        //                 this.text.visible = false;
        //                 this.title.visible = false;

        //         }else{
        //             this.sprite.visible = true;
        //             this.text.visible = true;
        //             this.title.visible = true;

        //         }
        //     }
        //     updatePosition(position){
        //         this.position = position;
        //         this.showRect = sceneGame.add.rectangle(position.x - showRange/2,position.y - showRange/2,this.showRect.width,this.showRect.height);
        //         this.sprite.x = position.x;
        //         this.sprite.y = position.y;
        //         this.title.x = position.x + 5;
        //         this.title.y = position.y;
        //         this.text.x = position.x + 5;
        //         this.text.y = position.y + fontSize + 10;

        //     }
        // }


        /*
        ++++++++++++++++++++++++++++++++++++
        Basic functions
        ++++++++++++++++++++++++++++++++++++
        */

        function zeroV() {
            return new Vector(0, 0);
        }


        function degToRad(deg) { // eslint-disable-line
            return Math.PI * deg / 180;
        }

        /*
        ++++++++++++++++++++++++++++++++++++
        Global Variables
        ++++++++++++++++++++++++++++++++++++
        */

        let isRunning = false;
        let isColliding = false;


        let canvasWidth = 1200;

        let canvasHeight = 1200;
        canvasWidth = $("#canvasWrapper").width() * window.devicePixelRatio;
        canvasHeight = $("#canvasWrapper").width() * window.devicePixelRatio;


        let ballradius = 40;

        let ball1, ball2, initAngle;

        const fontSize = 15; // eslint-disable-line 
        const showRange = 40; // eslint-disable-line 
        let borders, centreOfMass1, textScaleDown;

        let uiGroup;

        let arrowSprites = [];

        let traces = [];

        let angleSprite,dynamicSF,startingX,startingY;

        let d = new Date(); // eslint-disable-line no-unused-vars

        ball1 = {
            Lab: {
                name: "ball1Lab",
                spriteInstance: undefined,
                mass: vm.ball1Mass,
                initr: new Vector((canvasWidth / 2 - 100), (canvasHeight / 6)),
                v: new Vector(0, 0),
                initV: new Vector(0,0),
                postv: new Vector(0, 0)
            },
            CoM: {
                name: "ball1CoM",
                spriteInstance: undefined,
                mass: 1,
                initr: new Vector((canvasWidth / 2 - 100), (canvasHeight * 3 / 6)),
                v: new Vector(0, 0),
                postv: new Vector(0, 0)
            },
            radius: ballradius,
            color: 0xE9003A
        };
        ball2 = {
            Lab: {
                name: "ball2Lab",
                spriteInstance: undefined,
                mass: vm.ball2Mass,
                initr: new Vector((canvasWidth / 2 + 100), (canvasHeight / 6)),
                initV:new Vector(0,0),
                v: new Vector(0, 0),
                postv: new Vector(0, 0)
            },
            CoM: {
                name: "ball2CoM",
                spriteInstance: undefined,
                mass: 1,
                initr: new Vector((canvasWidth / 2 + 100), (canvasHeight * 3 / 6)),

                v: new Vector(0, 0),
                postv: new Vector(0, 0)
            },
            radius: ballradius,
            color: 0x00AEF2
        };
        ball1.CoM.mass = getReducedMass();
        ball2.CoM.mass = getReducedMass();

        /*
        ++++++++++++++++++++++++++++++++++++
        UI handling
        ++++++++++++++++++++++++++++++++++++
        */

        // $(document).ready(function(){
        //     $("#lodaingMessage").remove();
        // });

        function updateScatterAngle() {
            initAngle = degToRad(vm.collisionAngle);

            ball1.Lab.spriteInstance.y = ball1.Lab.initr.y + Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25;
            ball1.CoM.spriteInstance.y = ball1.CoM.initr.y + Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25;
            ball2.Lab.spriteInstance.y = ball2.Lab.initr.y - Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25;
            ball2.CoM.spriteInstance.y = ball2.CoM.initr.y - Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25;
        }

        // $(".inputs").each(function () {
        //     // To update the displayed value in HTML

        //     $(this).on('input', updateLabels);

        // });

        $('#vectorDrawEnable').on('click', updateLabels);

        function updateLabels() {
            console.log("updateLabels");
            $(".inputs").each(function () {
                // To update the displayed value in HTML
                $("#" + $(this).attr("id") + "Display").text($(this).val() + $("#" + $(this).attr("id") + "Display").attr("data-unit"));
            });
            if (!isRunning) {
                ball1.Lab.mass = vm.ball1Mass;
                ball2.Lab.mass = vm.ball2Mass;
                ball1.Lab.v.x = vm.velocityX;

                ball1.CoM.v = vCal(ball1.Lab.v, '-', getCoMV());
                ball2.CoM.v = vCal(ball2.Lab.v, '-', getCoMV());

                updateRadius(ball1);
                updateRadius(ball2);
                updateScatterAngle();
                centreOfMass1.x = getCoMR().x;
                centreOfMass1.y = getCoMR().y;

                ball1.CoM.spriteInstance.x = canvasWidth - 100 - getCoMR().x;
                ball1.CoM.spriteInstance.y += canvasHeight / 6 - getCoMR().y;
                ball2.CoM.spriteInstance.x = canvasWidth + 100 - getCoMR().x;
                ball2.CoM.spriteInstance.y += canvasHeight / 6 - getCoMR().y;

                recalculateVector();
            }
            
            // ball1.Lab.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
            // ball1.CoM.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
            // ball2.Lab.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
            // ball2.CoM.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
        }

        function updateRadius(ball) {
            ball.radius = 40 *  Math.sqrt(ball.Lab.mass);
            let newBallGraphic = sceneGame.add.graphics(0, 0);

            newBallGraphic.fillStyle(ball.color, 1);
            newBallGraphic.fillCircle(ball.radius, ball.radius, ball.radius);
            newBallGraphic.generateTexture(ball.Lab.name, ball.radius*2, ball.radius*2);
            let labSprite = sceneGame.add.sprite(ball.Lab.spriteInstance.x, ball.Lab.spriteInstance.y, ball.Lab.name);
            let CoMSprite = sceneGame.add.sprite(ball.CoM.spriteInstance.x, ball.CoM.spriteInstance.y, ball.Lab.name);
     
            ball.Lab.spriteInstance.destroy();
            ball.CoM.spriteInstance.destroy();
            ball.Lab.spriteInstance = labSprite;
            ball.CoM.spriteInstance = CoMSprite;

            ball.Lab.spriteInstance.setOrigin(0.5, 0.5);
            ball.CoM.spriteInstance.setOrigin(0.5, 0.5);
            ball.Lab.spriteInstance.setDepth(5);
            ball.CoM.spriteInstance.setDepth(5);
            newBallGraphic.destroy();
        }


        $("#runButton").on('click', function () {
            if (!isRunning) {
                isRunning = true;
                $("#runButton").val("Reset");
            } else if (isRunning) {
                //Run reset code
                isRunning = false;
                resetSimulation();
                $("#runButton").val("Run");
            }
            //console.log("Pre Collision KE:" + getLabKE().toString());
        });


        function resetSimulation() {
            ball1.Lab.spriteInstance.x = ball1.Lab.initr.x;
            ball1.Lab.spriteInstance.y = ball1.Lab.initr.y;
            ball2.Lab.spriteInstance.x = ball2.Lab.initr.x;
            ball2.Lab.spriteInstance.y = ball2.Lab.initr.y;

            ball1.CoM.spriteInstance.x = ball1.CoM.initr.x;
            ball1.CoM.spriteInstance.y = ball1.CoM.initr.y;
            ball2.CoM.spriteInstance.x = ball2.CoM.initr.x;
            ball2.CoM.spriteInstance.y = ball2.CoM.initr.y;

            clearVectors();

            ball1.CoM.spriteInstance.visible = true;
            ball2.CoM.spriteInstance.visible = true;
            ball1.Lab.spriteInstance.visible = true;
            ball2.Lab.spriteInstance.visible = true;

            isColliding = false;


            for (let i = 0; i < traces.length; i++) {
                traces[i].destroy();
            }
            traces = [];

            centreOfMass1.x = (canvasWidth / 2);
            centreOfMass1.y = (canvasHeight / 6);

            ball1.Lab.v.y = 0;
            ball2.Lab.v = zeroV();

            ball1.Lab.postv = zeroV();
            ball1.CoM.postv = zeroV();
            ball2.Lab.postv = zeroV();
            ball2.CoM.postv = zeroV();

            updateLabels();
        }

        /*
        ++++++++++++++++++++++++++++++++++++
        Functions based on Global letiables and basic functions
        ++++++++++++++++++++++++++++++++++++
        */

        function getReducedMass() {
            return (ball1.Lab.mass * ball2.Lab.mass) / (ball1.Lab.mass + ball2.Lab.mass);
        }

        function getCoMR() {
            return vCal(new Vector(ball1.Lab.mass * ball1.Lab.spriteInstance.x + ball2.Lab.mass * ball2.Lab.spriteInstance.x, ball1.Lab.mass * ball1.Lab.spriteInstance.y + ball2.Lab.mass * ball2.Lab.spriteInstance.y), '*', 1 / (ball1.Lab.mass + ball2.Lab.mass));
        }

        function getCoMV() {
            //return new vCal(new Vector(ball1.Lab.v.x*ball1.Lab.mass + ball2.Lab.v.x*ball2.Lab.mass,ball1.Lab.v.y*ball1.Lab.mass + ball2.Lab.v.y*ball2.Lab.mass),'*',1.0/(ball1.Lab.mass +ball2.Lab.mass));
            return vCal(vCal(vCal(ball1.Lab.v, "*", ball1.Lab.mass), "+", vCal(ball2.Lab.v, "*", ball2.Lab.mass)), "*", 1.0 / (ball1.Lab.mass + ball2.Lab.mass));
        }
 
        function getLabKE() { // eslint-disable-line no-unused-vars
            return 0.5 * ball1.Lab.mass * Math.pow(ball1.Lab.v.getMag(), 2) + 0.5 * ball2.Lab.mass * Math.pow(ball2.Lab.v.getMag(), 2);
        }

        function onCollision() {
            ball1.Lab.v = ball1.Lab.postv;
            ball2.Lab.v = ball2.Lab.postv;
            ball1.CoM.v = ball1.CoM.postv;
            ball2.CoM.v = ball2.CoM.postv;

            isColliding = true;
        }

        let WebFontConfig = { // eslint-disable-line no-unused-vars
            
            //  'active' means all requested fonts have finished loading
            //  We set a 1 second delay before calling 'createText'.
            //  For some reason if we don't the browser cannot render the text the first time it's created.
            active: function() { game.time.events.add(Phaser.Timer.SECOND, createText, this); }, // eslint-disable-line no-undef

            //  The Google Fonts we want to load (specify as many as you like in the array)
            google: {
            families: ['Fira Sans']
            }

        };


        /*
        ++++++++++++++++++++++++++++++++++++
        Phaser.io functions
        ++++++++++++++++++++++++++++++++++++
        */
        let config = {
            type: Phaser.AUTO,
            scale: {
                parent: "canvasWrapper",
                mode: Phaser.Scale.FIT,
                width: canvasWidth,
                height: canvasHeight,
            },
            physics: {
                default: 'p2',
            },
            scene: {
                preload: preload,
                create: create,
                update: update,
            },
            backgroundColor: "#EBEEEE",
        };

        console.log("Start game");

        var game = new Phaser.Game(config);
        var sceneGame;

        // let game = new Phaser.Game(canvasWidth, canvasHeight, Phaser.CANVAS, 'canvasWrapper', {
        //     preload: preload,
        //     create: create,
        //     update: update
        // });

        console.log("new Phaser.Game");

        function preload() {
            console.log("preload");
            if (window.devicePixelRatio >= 1.5) {
                this.load.image('cofmPNG', './assets/cofm.png');
                this.load.image('p1m1m2', 'assets/p1m1m2latex@2x.png');
                this.load.image('p1', 'assets/p1latex@2x.png');
                this.load.image('pstar', 'assets/pstarlatex@2x.png');
                this.load.image('qstar', 'assets/qstarlatex@2x.png');
                this.load.image('q1', 'assets/q1latex@2x.png');
                this.load.image('q2', 'assets/q2latex@2x.png');
                this.load.image('u1', 'assets/u1latex@2x.png');
                this.load.image('v1', 'assets/v1latex@2x.png');
                this.load.image('v2', 'assets/v2latex@2x.png');
            } else {
                this.load.image('cofmPNG', 'assets/cofm.png');
                this.load.image('p1m1m2', 'assets/p1m1m2latex.png');
                this.load.image('p1', 'assets/p1latex.png');
                this.load.image('pstar', 'assets/pstarlatex.png');
                this.load.image('qstar', 'assets/qstarlatex.png');
                this.load.image('q1', './assets/q1latex.png');
                this.load.image('q2', 'assets/q2latex.png');
                this.load.image('u1', 'assets/u1latex.png');
                this.load.image('v1', 'assets/v1latex.png');
                this.load.image('v2', 'assets/v2latex.png');
            }
            this.load.script('webfont', '//ajax.googleapis.com/ajax/libs/webfont/1.4.7/webfont.js');
            textScaleDown = window.devicePixelRatio;
        }

        function clearVectors() {

            for (let i = 0; i < arrowSprites.length; i++) {
                arrowSprites[i].destroy();
            }
            arrowSprites = [];
        }


        function create() {
            console.log("create");
            uiGroup = this.add.group();
            sceneGame = this;

            this.canvasWidth = $("#canvasWrapper").width() * window.devicePixelRatio;
            this.canvasHeight = $("#canvasWrapper").width() * window.devicePixelRatio;
            // this.scale.scaleMode = Phaser.ScaleManager.USER_SCALE;
            // this.scale.setUserScale(1 / window.devicePixelRatio, 1 / window.devicePixelRatio);
            // this.renderer.renderSession.roundPixels = true;
            // Phaser.Canvas.setImageRenderingCrisp(this.game.canvas);


            // this.stage.backgroundColor = "#EBEEEE";

            let ball1G = this.add.graphics(0, 0);
            let cofmad = this.add.sprite(500, 500, "cofmPNG");
            cofmad.setDepth(5);

            ball1G.fillStyle(0xE9003A, 1);
            ball1G.fillCircle(ball1.radius, ball1.radius, ball1.radius);
            ball1G.generateTexture("ball1G", ball1.radius*2, ball1.radius*2);

            
            ball1.Lab.spriteInstance = this.add.sprite(ball1.Lab.initr.x, ball1.Lab.initr.y, "ball1G");
            ball1.CoM.spriteInstance = this.add.sprite(ball1.CoM.initr.x, ball1.CoM.initr.y, "ball1G");
            ball1.Lab.spriteInstance.setOrigin(0.5, 0.5);
            ball1.CoM.spriteInstance.setOrigin(0.5, 0.5);
            ball1G.destroy();

            let ball2G = this.add.graphics(0, 0);

            ball2G.fillStyle(0x00AEF2, 1);
            ball2G.fillCircle(ball2.radius, ball2.radius, ball2.radius);
            ball2G.generateTexture("ball2G", ball2.radius*2, ball2.radius*2);

            ball2.CoM.spriteInstance = this.add.sprite(ball2.CoM.initr.x, ball2.CoM.initr.y, "ball2G");
            ball2.Lab.spriteInstance = this.add.sprite(ball2.Lab.initr.x, ball2.Lab.initr.y, "ball2G");
            ball2.Lab.spriteInstance.setOrigin(0.5, 0.5);
            ball2.CoM.spriteInstance.setOrigin(0.5, 0.5);
            ball2G.destroy();


            let bordersG = this.add.graphics((canvasWidth / 2), (canvasHeight / 2));

            bordersG.lineStyle(6 * textScaleDown, 0x003E74, 1);
            bordersG.beginPath();
            bordersG.moveTo(0, 0);
            bordersG.lineTo(0, (canvasHeight ));
            bordersG.lineTo((canvasWidth ), (canvasHeight));
            bordersG.lineTo((canvasWidth), 0);
            bordersG.lineTo(0, 0);

            bordersG.lineStyle(3 * textScaleDown, 0x003E74, 1);
            bordersG.moveTo(0, (canvasHeight / 3));
            bordersG.lineTo((canvasWidth), (canvasHeight / 3));
            bordersG.moveTo(0, (canvasHeight * 2 / 3));
            bordersG.lineTo((canvasWidth ), (canvasHeight * 2 / 3));
            bordersG.closePath();
            bordersG.strokePath();

            bordersG.generateTexture("bordersG", canvasWidth, canvasHeight);

            borders = this.add.sprite((canvasWidth / 2), (canvasHeight / 2), "bordersG");
            borders.setOrigin(0.5, 0.5);

            bordersG.destroy();

            let style = {
                font: (18 * textScaleDown) + "px Fira Sans",
                fill: "#003E74",
                wordWrap: false,
                align: "left",
                backgroundColor: "#EBEEEE"
            };

            let labFrameText = this.add.text(16, 10, "Lab Frame", style);
            labFrameText.setOrigin(0, 0);
            uiGroup.add(labFrameText);

            let centreOfMassFrameText = this.add.text(16, ( canvasHeight / 3 + 10 ), "Centre of Mass Frame", style);
            centreOfMassFrameText.setOrigin(0, 0);
            uiGroup.add(centreOfMassFrameText);

            let vectorDiagramText = this.add.text(16, ( canvasHeight * 2 / 3 + 10 ), "Vector Diagram", style);
            vectorDiagramText.setOrigin(0, 0);
            uiGroup.add(vectorDiagramText);

            centreOfMass1 = this.add.sprite((canvasWidth / 2), (canvasHeight  / 6), 'cofmPNG');
            centreOfMass1.setDepth(1);
            centreOfMass1.setOrigin(0.5, 0.5);

            let centreOfMass2 = this.add.sprite(canvasWidth / 2, canvasHeight / 2, "cofmPNG");
            centreOfMass2.setDepth(1);
            centreOfMass2.setOrigin(0.5, 0.5);

            updateLabels();
        }


        let t = 0;

        function update() { 
            console.log("update");
            // this.world.sort('z',Phaser.Group.SORT_ASCENDING);
            updateLabels();

            if (isRunning){
                ball1.Lab.spriteInstance.x += ball1.Lab.v.x;
                ball1.Lab.spriteInstance.y -= ball1.Lab.v.y;
                ball2.Lab.spriteInstance.x += ball2.Lab.v.x;
                ball2.Lab.spriteInstance.y -= ball2.Lab.v.y;


                if ((t % (ball1.Lab.v.getMag() * 3)) < 1) {
                    addTrace(ball1.Lab);
                }
                if ((t % (ball2.Lab.v.getMag() * 3)) < 1) {
                    addTrace(ball2.Lab);
                }
                if ((t % (ball1.CoM.v.getMag() * 3)) < 1) {
                    addTrace(ball1.CoM);
                }
                if ((t % (ball2.CoM.v.getMag() * 3)) < 1) {
                    addTrace(ball2.CoM);
                }

                centreOfMass1.x = getCoMR().x;
                centreOfMass1.y = getCoMR().y;

                //Centre of mass motion

                ball2.CoM.spriteInstance.x += ball2.CoM.v.x;
                ball2.CoM.spriteInstance.y -= ball2.CoM.v.y;
                ball1.CoM.spriteInstance.x += ball1.CoM.v.x;
                ball1.CoM.spriteInstance.y -= ball1.CoM.v.y;

                // $('#ball1LabVelocityX').prop("disabled", true);
                // $('#ball1LabMass').prop("disabled", true);
                // $('#ball2LabMass').prop("disabled", true);
                // $('#ballCollisionAngle').prop("disabled", true);
                // $('#vectorDrawEnable').prop("disabled", true);

                // let velocityDisplay = [ball1.Lab.v.x.toFixed(2), ball1.Lab.v.y.toFixed(2), ball2.Lab.v.x.toFixed(2), ball2.Lab.v.y.toFixed(2)];

                // let i = 0;
                // $(".velocityDisplay").each(function () {
                //     $(this).text(velocityDisplay[i] + $(this).attr("data-unit"));
                //     i++;
                // });

                if(ball1.Lab.spriteInstance.y + ball1.radius/2 > canvasHeight/3) ball1.Lab.spriteInstance.visible = false;
                if(ball2.Lab.spriteInstance.y + ball2.radius/2 > canvasHeight/3) ball2.Lab.spriteInstance.visible = false;
                if(ball1.CoM.spriteInstance.y - ball1.radius/2 < canvasHeight/3) ball1.CoM.spriteInstance.visible = false;
                if(ball1.CoM.spriteInstance.y + ball1.radius/2 > canvasHeight*2/3) ball1.CoM.spriteInstance.visible = false;
                if(ball2.CoM.spriteInstance.y  - ball2.radius/2 < canvasHeight/3) ball2.CoM.spriteInstance.visible = false;
                if(ball2.CoM.spriteInstance.y + ball2.radius/2 > canvasHeight*2/3) ball2.CoM.spriteInstance.visible = false;

                if (!isColliding && Math.pow(ball1.Lab.spriteInstance.x - ball2.Lab.spriteInstance.x,2) + Math.pow(ball1.Lab.spriteInstance.y - ball2.Lab.spriteInstance.y,2) <= Math.pow(ball1.radius+ball2.radius,2)/4) {
                    onCollision();
                }

                // ball1.Lab.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
                // ball1.CoM.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
                // ball2.Lab.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
                // ball2.CoM.spriteInstance.blendMode = PIXI.blendModes.DARKEN;
            }
            t += 1;
        }

        function drawArrow(originV, vectorV, rectIndex, color = 0x006EAF, latexID = "") {
            // let origin,vector;
            let scaleFactor;
            let disStarting = 1;
            if(rectIndex === 3) {
                // vector = new Vector(vectorV.x * 7 /dynamicSF,-1*vectorV.y*7/dynamicSF);
                // if(latexID === 'p1'){
                //     origin = new Vector(originV.x,-1*originV.y);
                // }
                // else{
                //      origin = new Vector(originV.x * 7/dynamicSF,-1*originV.y * 7 /dynamicSF);
                // }
                // // Flip directions for canvas y-axis

                scaleFactor = dynamicSF;
            }else{
                // vector = new Vector(vectorV.x,-1*vectorV.y);
                // origin = new Vector(originV.x, -1 * originV.y);

                scaleFactor = canvasWidth / 16;
                disStarting = 0;
            }

            let arrowG = sceneGame.add.graphics(0,0);

            let mag = vectorV.getMag() * scaleFactor;
            console.log(mag);

            arrowG.lineStyle(2, color, 1);
            arrowG.beginPath();
            arrowG.moveTo(0, 0);
            arrowG.lineTo(0, 4);
            arrowG.lineTo(0, -4);
            arrowG.lineTo(0, 0);
            arrowG.lineTo(mag, 0);

            for (var i = 1; i <= Math.floor(vectorV.getMag()); i++) {
                arrowG.lineTo(i*scaleFactor, 0);
                arrowG.lineTo(i*scaleFactor, 4);
                arrowG.lineTo(i*scaleFactor, -4);
                arrowG.lineTo(i*scaleFactor, 0);
            }    

            arrowG.lineTo(mag, 0);
            arrowG.lineTo(mag, 4);
            arrowG.lineTo(mag, -4);
            arrowG.lineTo(mag, 0);



            // arrowG.beginPath();
            arrowG.moveTo(mag / 2, 0);
            arrowG.lineTo(mag / 2, 0);
            arrowG.lineTo(mag / 2, 6);
            arrowG.lineTo(mag / 2 + 10, 0);
            arrowG.lineTo(mag / 2, -6);
            arrowG.lineTo(mag / 2, 0);
            arrowG.closePath();
            arrowG.stroke();
            arrowG.generateTexture("arrow");
            
            // arrowG.endPath();


            arrowSprites.push(sceneGame.add.sprite((canvasWidth / 2 + startingX*disStarting + originV.x * scaleFactor), (canvasHeight * (rectIndex * 2 - 1) / 6 + startingY*disStarting - originV.y * scaleFactor), "arrow"));
            arrowSprites[arrowSprites.length - 1].setOrigin(0, 0.5);
            arrowSprites[arrowSprites.length - 1].rotation = -1*vectorV.getArg();
            console.log(-1*vectorV.getArg());

            if (latexID !== "") {
                arrowSprites.push(sceneGame.add.sprite((canvasWidth / 2 + startingX*disStarting + originV.x * scaleFactor + vectorV.x * scaleFactor / 2), (canvasHeight * (rectIndex * 2 - 1) / 6 + startingY*disStarting - originV.y * scaleFactor - vectorV.y * scaleFactor / 2), latexID));
                arrowSprites[arrowSprites.length - 1].setOrigin(0.5, 0);
            }

            arrowG.destroy();
        }

        function drawAngle(vecta, vectb, pos, color) { // eslint-disable-line no-unused-vars
            let arcG = sceneGame.add.graphics(0, 0);
            arcG.lineStyle(8, color);
            arcG.arc(0, 0, 50, Math.max(vecta.getArg(),vectb.getArg()), Math.min(vecta.getArg(),vectb.getArg()), true);
            arcG.strokePath();
            arcG.generateTexture("arc");
            if(angleSprite != null) angleSprite.destroy();
            angleSprite =  sceneGame.add.sprite(pos.x + canvasWidth/2,pos.y + canvasHeight/6, "arc");
            angleSprite.setOrigin(0,0.5);
            arcG.destroy();
        }

        function getLabP() { // eslint-disable-line no-unused-vars
            return new Vector(ball1.Lab.v.x* ball1.Lab.mass + ball2.Lab.v.x*ball2.Lab.mass,ball2.Lab.v.y*ball2.Lab.mass+ball1.Lab.mass*ball1.Lab.v.y);
        }

        function recalculateVector() {
            console.log("recalculateVector");
            let results = doPhysics({mass:ball1.Lab.mass,initV:ball1.Lab.v},{mass:ball2.Lab.mass,initV:ball2.Lab.v},initAngle);


            ball1.CoM.postv = results[0];
            ball2.CoM.postv = results[1];
            let p1 = vCal(ball1.Lab.v, "*", ball1.Lab.mass);

            let coMV = getCoMV();
            ball1.Lab.postv = vCal(ball1.CoM.postv, '+', coMV);
            ball2.Lab.postv = vCal(ball2.CoM.postv, '+', coMV);

            let q1 = vCal(ball1.Lab.postv, "*", ball1.Lab.mass);
            let q2 = vCal(ball2.Lab.postv, "*", ball2.Lab.mass);

            let pStar = vCal(vCal(ball2.Lab.v, '-', ball1.Lab.v), '*', getReducedMass());
            let q2Star = vCal(vCal(ball2.Lab.postv,'-',ball1.Lab.postv),'*',getReducedMass());
            let q1Star = vCal(q2Star,'*',-1);

            dynamicSF = 1;

            if( ( p1.getMag()/Math.abs(q1.y)) <= ( (canvasWidth*0.6) / (canvasHeight/3*0.6) ) ){
                // fit to Math.abs(q1.y)
                dynamicSF = (canvasHeight/3*0.6)/(Math.abs(q1.y)+0.4);
            }else if( ( p1.getMag()/Math.abs(q1.y)) > ( (canvasWidth*0.6) / (canvasHeight/3*0.6) ) ){
                // fit to p1.getMag()
                dynamicSF = (canvasWidth*0.6)/p1.getMag();
            }

            startingX = -1*(dynamicSF*p1.getMag()/2);
            startingY = (dynamicSF*(q1.y-0.4)/2);

            // if('comLab' in toolTips){
            //     toolTips['comLab'].updatePosition(getCoMR());
            // }else {
            //     toolTips['comLab'] = new ToolTip(new Vector(getCoMR().x,getCoMR().y),"Centre of Mass","The single point in the system where\n all the total mass of the object can\n be taken to act at.");
            // }

            clearVectors();

            if ($('#vectorDrawEnable').is(':checked')) {
                console.log("Checked");

                let scalefactor = canvasWidth / 16;
                let colPoint = 100-ball2.radius;

                drawArrow(vCal(new Vector(colPoint/scalefactor, 0 - (Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25)/scalefactor ),'-',ball1.Lab.v), ball1.Lab.v, 1, 0xE40043, "u1");
                drawArrow(new Vector(colPoint/scalefactor, 0 - (Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25)/scalefactor ), ball1.Lab.postv, 1, 0xD24000, "v1");
                drawArrow(new Vector((ball2.Lab.spriteInstance.x-canvasWidth*0.5)/scalefactor, (Math.sin(initAngle)*(ball1.radius + ball2.radius)*0.25)/scalefactor),ball2.Lab.postv, 1, 0x00ACD7, "v2");
                //CoM frame
                drawArrow(new Vector(0-pStar.x ,0),pStar,2,0x960078,'pstar');
                drawArrow(new Vector(pStar.x,0),vCal(pStar,'*',-1),2,0x960078,'pstar');
                drawArrow(new Vector(0,0),q1Star,2,0x960078,'qstar');
                drawArrow(new Vector(0,0),q2Star,2,0x960078,'qstar');
            }

            drawArrow(zeroV(), q1, 3, 0xDD2501, "q1");
            drawArrow(q1, q2, 3, 0x0091D4, 'q2');
            drawArrow(zeroV(), vCal(vCal(pStar,'*',-1), "*", (ball1.Lab.mass / ball2.Lab.mass)), 3, 0xEC7300, "p1m1m2");
            drawArrow(vCal(vCal(pStar,'*',-1), "*", (ball1.Lab.mass / ball2.Lab.mass)),vCal(pStar,'*',-1), 3, 0x960078, "pstar");
            drawArrow(vCal(vCal(pStar,'*',-1), "*", (ball1.Lab.mass / ball2.Lab.mass)), q1Star, 3, 0x960078, "qstar");
            drawArrow(new Vector(0, -0.4), p1, 3, 0xE40043, "p1");
        }

        function addTrace(ball) {
            let ballG = sceneGame.add.graphics(0, 0);
            let color;
            if (ball.name === "ball1Lab" || ball.name === "ball1CoM") {
                color = 0xE9003A;
            } else if (ball.name === "ball2Lab" || ball.name === "ball2CoM") {
                color = 0x00AEF2;
            }

            if(ball.spriteInstance.visible){
                ballG.lineStyle(1, color, 1);
                ballG.fillStyle(color, 1);
                ballG.fillCircle(7, 7, 7);
                ballG.generateTexture("trace");

                traces.push(sceneGame.sprite.add(ball.spriteInstance.x, ball.spriteInstance.y, "trace"));
                traces[traces.length -1].setOrigin(0.5,0.5);
                ballG.destroy();
            }
        }

        console.log("End");
        
    },
}
</script>
<style>
.center{
    display:flex;
    flex-direction: column;
    align-items: center;
}
</style>