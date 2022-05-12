<template>
    <div id="canvas" class="w-screen h-screen"></div>
    <div class="absolute top-5 left-5 flex flex-col gap-4">
        <button @click="toggleInfo()">
            <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" role="img" class="w-12 transition hover:opacity-80 iconify iconify--bi" preserveAspectRatio="xMidYMid meet" viewBox="0 0 16 16"><path fill="black" d="M2 0a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V2a2 2 0 0 0-2-2H2zm3.496 6.033a.237.237 0 0 1-.24-.247C5.35 4.091 6.737 3.5 8.005 3.5c1.396 0 2.672.73 2.672 2.24c0 1.08-.635 1.594-1.244 2.057c-.737.559-1.01.768-1.01 1.486v.105a.25.25 0 0 1-.25.25h-.81a.25.25 0 0 1-.25-.246l-.004-.217c-.038-.927.495-1.498 1.168-1.987c.59-.444.965-.736.965-1.371c0-.825-.628-1.168-1.314-1.168c-.803 0-1.253.478-1.342 1.134c-.018.137-.128.25-.266.25h-.825zm2.325 6.443c-.584 0-1.009-.394-1.009-.927c0-.552.425-.94 1.01-.94c.609 0 1.028.388 1.028.94c0 .533-.42.927-1.029.927z"></path></svg>
        </button>
        <span>
            <span>3D: </span>
            <input type="checkbox" @click="toggleOrbitControls()">
        </span>
        <button class="add" @click="newBox()">Add Box +</button>
        <button class="add" @click="newLight()">Add Light +</button>
    </div>
    <div v-show="info" class="com top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 p-12 w-fit rounded border border-white">
        <h1 class="text-xl font-bold">Optic Simulator</h1> <br>
        <div class="mb-5">
            <span class="text-lg">Usage:</span> <br>
            * Add point lights and boxes from sidebar. <br>
            * By drag'n'drop 3D objects, you can position on the plane. <br>
            * By enabling 3D mode from sidebar you can zoom and rotate the plane. <br>
            <span>Open Source on: <a class="underline" href="https://github.com/orhanemree/threejs-optic">GitHub</a></span> <br>
        </div>
        <span>Made with <mark>Vue</mark> and <mark>Three</mark> by <a class="underline" href="https://github.com/orhanemree">orhanemree</a></span>
    </div>
    <div v-if="mobileWarn" class="com inset-0 text-xl">
        <span>This simulation does not support mobile devices yet.</span>
        <a class="underline" href="https://github.com/orhanemree/threejs-optic">GitHub</a>
    </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { DragControls } from "three/examples/jsm/controls/DragControls.js";

export default {
    name: "Model",
    data(){
        return{
            info: false,
            mobileWarn: false,
            draggables: [],
        }
    },
    methods: {
        init(){

            let geometry, material;

            // three.js setup
            const canvas = document.getElementById("canvas");
            if (canvas.clientWidth < 768){
                this.mobileWarn = true;
            }
            this.camera = new THREE.PerspectiveCamera(75, canvas.clientWidth / canvas.clientHeight, 0.1, 10000);
            this.camera.position.y = 35;
            this.scene = new THREE.Scene();
            this.camera.lookAt(this.scene.position);

            // renderer
            this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            this.renderer.setSize(canvas.clientWidth, canvas.clientHeight);
            this.renderer.shadowMap.enabled = true;
            canvas.appendChild(this.renderer.domElement);

            // ambient light
            const ambientLight = new THREE.AmbientLight(0x111111, 1);
            this.scene.add(ambientLight);

            // plane
            geometry = new THREE.PlaneGeometry(50, 50);
            material = new THREE.MeshStandardMaterial({color: 0xcccccc, side: THREE.DoubleSide});
            const plane = new THREE.Mesh(geometry, material);
            plane.rotation.x = Math.PI / 2;
            plane.receiveShadow = true;
            this.scene.add(plane);

            geometry = new THREE.PlaneGeometry(50, 20);
            const wall1 = new THREE.Mesh(geometry, material);
            wall1.position.z = 25;
            wall1.position.y = 10;
            wall1.receiveShadow = true;
            this.scene.add(wall1);

            const wall2 = new THREE.Mesh(geometry, material);
            wall2.rotation.y = Math.PI / 2;
            wall2.position.x = 25;
            wall2.position.y = 10;
            wall2.receiveShadow = true;
            this.scene.add(wall2);

            this.pointLights = 0;

            // drag control
            this.drag = new DragControls(this.draggables, this.camera, this.renderer.domElement);
            this.drag.addEventListener("drag", e => {
                e.object.position.y = e.object.geometry.type === "BoxGeometry" ? 1.5 : 3;
            });

            // orbit controls
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enabled = false;
            this.controls.enablePan = false;

            console.log(this.scene);
        },
        animate(){
            requestAnimationFrame(this.animate);
            this.controls.update();
            this.renderer.render(this.scene, this.camera);
            for (let i = 0; i < this.scene.children.length; i++){
                if (this.scene.children[i].type === "PointLight"){
                    this.scene.children[i].position.x = this.scene.children[i + 1].position.x;
                    this.scene.children[i].position.z = this.scene.children[i + 1].position.z;
                }
            }
        },
        toggleInfo(){
            this.info = !this.info;
        },
        toggleOrbitControls(){
            if (!this.info){
                this.camera.position.set(0, 35, 0);
                this.controls.enabled = !this.controls.enabled;
                this.drag.enabled = !this.drag.enabled;
            }
        },
        newBox(){
            if (!this.info){
                const box = new THREE.Mesh(
                    new THREE.BoxGeometry(3, 3, 3),
                    new THREE.MeshPhongMaterial({color: 0xab0c00})
                );
                box.castShadow = true;
                box.receiveShadow = true;
                box.position.y = 1.5;
                this.scene.add(box);
                this.draggables.push(box);
            }
        },
        newLight(){
            if (!this.info){
                // point light
                const pointLight = new THREE.PointLight(0xcccccc, 1);
                pointLight.position.set(0, 3, 0);
                pointLight.castShadow = true;
                this.scene.add(pointLight);
    
                // sphere
                const sphere = new THREE.Mesh(
                    new THREE.SphereGeometry(2, 20, 10),
                    new THREE.MeshBasicMaterial({color: 0xf0f0f0})
                );
                sphere.position.y = 3;
                this.scene.add(sphere);
                this.draggables.push(sphere);
    
                this.pointLights++;
            }
        }
    },
    mounted(){
        this.init();
        this.animate();
    }
}
</script>