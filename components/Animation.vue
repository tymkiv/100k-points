<template>
    <div class="animation" ref="container">

    </div>
</template>

<script>
import {mapState} from 'vuex';
import * as THREE from 'three';
import gsap from 'gsap';

import VTKLoader from '~/lib/vtkloader';

const OrbitControls = require('three-orbit-controls')(THREE);
 
export default {
    data() {
        return {
            manager: {},
            states: [],
        }
    },
    watch: {
        page: function(newValue, oldValue) {
            // console.dir(manager);
            console.log(this.page);
            this.manager.change(newValue, oldValue);
            // console.log(newValue, oldValue);

        }
    },
    mounted() {
        class Sketch {
            constructor(container, startState) {
                this.container = container;
                this.startState = startState;

                this.init();
                // Do something
                this.states = {};
                this.vtkLoader.load('/female.vtk', geometry => {
                // this.geometry = geometry;
                this.geometry = new THREE.BufferGeometry();
                this.count = geometry.attributes.position.count;

                this.girlArr = Float32Array.from(geometry.attributes.position.array); 
                this.girlAttr = new THREE.BufferAttribute( this.girlArr, 3 );
                
                this.cubeArr = new Float32Array(this.count * 3);
                this.randomArr = new Float32Array(this.count * 3);
                
                for (let i = 0; i < this.count * 3; i += 3) {
                    this.cubeArr[i + 0] = Math.random() * 2;
                    this.cubeArr[i + 1] = Math.random() * 2 + 1;
                    this.cubeArr[i + 2] = Math.random() * 2 - 1;

                    this.randomArr[i + 0] = Math.random() * 20;
                    this.randomArr[i + 1] = Math.random() * 20 - 10;
                    this.randomArr[i + 2] = Math.random() * 20 - 10;
                }

                this.cubeAttr = new THREE.BufferAttribute( this.cubeArr, 3 );
                this.randomAttr = new THREE.BufferAttribute( this.randomArr, 3 );
                
                this.states['index'] = this.girlAttr;
                this.states['about'] = this.cubeAttr;
                this.states['random'] = this.randomAttr;
                // this.states.push(this.girlAttr); 
                // this.states.push(this.cubeAttr); 
                // console.log(this.states);
                this.geometry.setAttribute('position', this.randomAttr);
                this.geometry.setAttribute('source', this.randomAttr);
                this.geometry.setAttribute('target', this.states[this.startState]);

                this.material = new THREE.RawShaderMaterial({
                    uniforms: {
                    dot: { type: 't', value: new THREE.TextureLoader().load('/reddot.png') },
                    blend: { type: 'f', value: 0 },
                    size: { type: 'f', value: window.devicePixelRatio },
                    dimensions: { type: 'v2', value: new THREE.Vector2(this.nW, this.nH) }
                    },
                    vertexShader: `
                    precision highp float;
                    attribute vec3 position;
                    attribute vec3 source;
                    attribute vec3 target;

                    uniform mat4 modelViewMatrix;
                    uniform mat4 projectionMatrix;

                    uniform float size;
                    uniform float blend;
                    uniform sampler2D sourceTex;
                    uniform sampler2D targetTex;
                    uniform vec2 dimensions;

                    varying vec3 vColor;
                    void main() {

                        vColor = vec3(1., 1., 1.);
                        
                        vec3 p = mix(source, target, blend);

                        vec4 mvPosition = modelViewMatrix * vec4(p, 1.);
                        gl_PointSize = size * (1. / - mvPosition.z);
                        gl_Position = projectionMatrix * mvPosition;

		            }`,
                    fragmentShader: `
                    precision highp float;
                    varying vec3 vColor;
                    uniform vec2 u_resolution;
                    uniform vec2 u_mouse;
                    uniform float u_time;
                    uniform sampler2D dot;

                    void main() {
                        vec2 st = gl_FragCoord.xy/u_resolution;           
                        vec4 color = texture2D(dot, gl_PointCoord);
                        gl_FragColor = color;
                    }`,
                    alphaTest: 0.5,
                    transparent: true,
                });

                const points = new THREE.Points(this.geometry, this.material);
                this.scene.add(points);

                points.position.y = -2;
                points.rotation.y = -1.6;

                this.change(this.startState, 'random', 3);

                // document.querySelector('body').addEventListener('click', () => {
                //     gsap.to(this.material.uniforms.blend, {value: 1, duration: 0.5, onComplete: () => {
                //     const tmp1 = this.geometry.attributes.target;
                //     this.material.uniforms.blend.value = 0;
                //     this.geometry.attributes.target = this.geometry.attributes.source;
                //     this.geometry.attributes.source = tmp1;

                //     }})
                // })
                });
                
                // End of something

                
                this.animate();
            }

            change(newValue, oldValue, duration = 1.5) {
                this.geometry.attributes.source = this.states[oldValue];
                this.geometry.attributes.target = this.states[newValue];
                this.material.uniforms.blend.value = 0;
                gsap.to(this.material.uniforms.blend, {value: 1, duration: duration})
            }

            // changeOnComplete() {
            //     const tmp1 = this.geometry.attributes.target;
            //     this.material.uniforms.blend.value = 0;
            //     this.geometry.attributes.target = this.geometry.attributes.source;
            //     this.geometry.attributes.source = tmp1;
            // }

            init() {
                this.camera = new THREE.PerspectiveCamera(
                70,
                this.container.clientWidth / this.container.clientHeight,
                0.001, 500
                );
                this.camera.position.set( 0, 0, 4 );

                this.scene = new THREE.Scene();

                this.renderer = new THREE.WebGLRenderer();
                this.renderer.setPixelRatio( window.devicePixelRatio );

                this.container.appendChild( this.renderer.domElement );

                this.vtkLoader = new THREE.VTKLoader();

                this.controls = new OrbitControls(this.camera, this.renderer.domElement);

                this.onWindowResize();

                window.addEventListener('resize', this.onWindowResize.bind(this));
            }

            render() {
                this.renderer.render(this.scene, this.camera);
            }

            animate() {
                this.render();
                requestAnimationFrame(this.animate.bind(this));
            }

            onWindowResize() {
                this.width = this.container.clientWidth;
                this.height = this.container.clientHeight;

                this.renderer.setSize(this.width, this.height);

                this.resolutionWidth = this.renderer.domElement.width;
                this.resolutionHeight = this.renderer.domElement.height;
            }
            }

            this.manager = new Sketch(this.$refs.container, this.page);
    }, 
    computed: mapState(['page']),
}
</script>

<style scoped>
    .animation {
        position: fixed;
        top: 0;
        left: 0;
        /* z-index: 2000; */
        z-index: -1;
        width: 100%;
        height: 100vh;
        pointer-events: none;
    }
</style>