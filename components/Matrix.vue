<template>
    <div class="fixed top-0 w-full h-full z-10">
        <div ref="container" class="h-full w-full"></div>

    </div>
</template>

<script setup>
import * as THREE from "three";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls.js"

const container = ref(null);

onMounted(()=>{
 const canvas = document.createElement('canvas');

 const ctx = canvas.getContext('2d');
 const fontSize = 20;
 const width = window.innerWidth * 2;
 const height = window.innerHeight * 2;
 const columns = Math.floor(width / fontSize);
 const rows = Math.floor(height / fontSize);



const words = ["Hello", "World","HTML","CSS","Javascript","Developer","Frontend","Backend","Fullstack","Three.js","Developer"];
const drawText =Array(columns).fill().map(()=>Array(rows).fill(""));
const drawStatus = Array(columns).fill().map(()=>Array(rows).fill(0));

canvas.width = width;
canvas.height = height;


function randomizeColumn(col){
    for (let y =0; y< rows; y++){
        drawText[col][y] = words[Math.floor(Math.random() * words.length)];
    }
}

function randomizeStart(){
    if(Math.random()>0){
        const column = Math.floor(Math.random() * columns);
        if(drawStatus[column].every(status => status === 0)){
           drawStatus[column][0]=100;
        }
    }
}

const fallSpeed = 1;
let frameCounter = 0;


function drawRandom(){
    ctx.fillStyle = `rgba(0,0,0,0.1)`;
    ctx.fillRect(0,0,width,height);

    for (let x = 0; x<columns; x++){
        for (let y=0; y<rows; y++){
            if(drawStatus[x][y]===100)
        {
            if (frameCounter % fallSpeed === 0){
                drawStatus[x][y] = 0;
                ctx.font = `${fontSize}px Monospace`;
                ctx.fillStyle = "rgba(0,255,0, 0.9)";
                ctx.fillText(drawText[x][y], x * fontSize, y * fontSize);

                if (y + 1 < rows){
                    drawStatus[x][y + 1] = 100;
                } else {
                    randomizeColumn(x);
                }
                 
            } 
            break;
        }
        }
    }
    frameCounter++;
}

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 10000);
camera.position.set(0, -1000, 0);
scene.background = new THREE.Color(0x000000);

const renderer = new THREE.WebGLRenderer({antialias: true});
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
container.value.appendChild(renderer.domElement);

const textureCanvas = new THREE.Texture(canvas);
const materialCanvas = new THREE.MeshLambertMaterial({map: textureCanvas, transparent: true, opacity: 1});
const geometryCanvas = new THREE.BoxGeometry(width, height, 1);
const meshCanvas = new THREE.Mesh(geometryCanvas, materialCanvas);
scene.add(meshCanvas);
meshCanvas.position.set(0, 0, 0);
meshCanvas.rotateX(1.6);
camera.lookAt(meshCanvas.position);

const lightAmb = new THREE.HemisphereLight(0xffffbb, 0x080820, 1);
lightAmb.position.set(0, 0, 0);
scene.add(lightAmb);

const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 0, 0);
controls.update();
controls.enableZoom = false;

function animate(){
    requestAnimationFrame(animate);
    randomizeStart();
    drawRandom();
    textureCanvas.needsUpdate = true;
    renderer.render(scene, camera);
}
animate();
 window.addEventListener('resize', ()=>{
     camera.aspect = window.innerWidth / window.innerHeight;
     camera.updateProjectionMatrix();
     renderer.setSize(window.innerWidth, window.innerHeight);
 });



});


</script>

