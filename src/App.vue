<script setup>
// 引入three.js
import * as THREE from 'three';
// 引入轨道控制器扩展库OrbitControls.js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';


// 虚拟场景 + 虚拟相机 -》渲染器（执行渲染操作）-》投影图（渲染结果）

// width和height用来设置Three.js输出的Canvas画布尺寸(像素px)
const width = 800; //宽度
const height = 500; //高度

// 1. 虚拟场景
//场景：创建3D场景对象Scene
const scene = new THREE.Scene()
//形状：创建一个长方体几何对象Geometry 设置几何体长宽高，也就是x、y、z三个方向的尺寸
const geometry = new THREE.BoxGeometry(50, 50, 50)
//外观：创建一个材质对象Material  MeshBasicMaterial-不会受到光照影响
const material = new THREE.MeshLambertMaterial({
  color: 0xff0000,//0xff0000设置材质颜色为红色
  transparent:true,//材质半透明设置:开启透明
  opacity:0.5,//材质半透明设置:设置透明度
})
//物体：网格模型Mesh,两个参数分别为几何体geometry、材质material
const mesh = new THREE.Mesh(geometry,material)
// 模型位置：.position  设置网格模型在三维空间中的位置坐标，默认是坐标原点
// mesh.position.set(0,10,0)
mesh.position.set(0,0,0)
// 需要通过.add()方法，把网格模型mesh添加到三维场景scene中。
scene.add(mesh)

//////////点光源：两个参数分别表示光源颜色和光照强度
// 参数1：0xffffff是纯白光,表示光源颜色
// 参数2：1.0,表示光照强度，可以根据需要调整
const pointLight = new THREE.PointLight(0xffffff, 1.0);
pointLight.decay = 0.0;//设置光源不随距离衰减，默认2.0
// 你可以对比不同光照强度明暗差异(传播同样距离)
pointLight.intensity = 10000.0;//光照强度
// pointLight.intensity = 50000.0;//光照强度
//点光源位置
pointLight.position.set(400, 200, 300);
scene.add(pointLight); //点光源添加到场景中

////////// AxesHelper：辅助观察的坐标系
const axesHelper = new THREE.AxesHelper(150);
scene.add(axesHelper);




////////////////////////

// 2. 虚拟相机
// 实例化一个透视投影相机对象 PerspectiveCamera( fov, aspect, near, far )
// foc:相机视锥体竖直方向视野角度; default=50
// aspect:相机视锥体水平方向和竖直方向长度比，一般设置为Canvas画布宽高比width / height; default=1
// near:相机视锥体近裁截面相对相机距离; default=0.1
// far:相机视锥体远裁截面相对相机距离，far-near构成了视锥体高度方向; default=2000
const camera = new THREE.PerspectiveCamera(30, width / height, 1, 3000);
// 相机位置:相机在Three.js三维坐标系中的位置  根据需要设置相机位置具体值
camera.position.set(500,500,500)
// camera.position.set(-1000, 0, 0);
// 相机观察目标:相机观察目标指向Threejs 3D空间中某个位置
// camera.lookAt(-800, 0, 0);
// camera.lookAt(0, 0, 0); //坐标原点
// camera.lookAt(0, 10, 0);  //y轴上位置10
camera.lookAt(mesh.position);//指向mesh对应的位置

////////////////////////

// 3. 渲染器
// 创建渲染器对象
const renderer = new THREE.WebGLRenderer()
// 设置Canvas画布尺寸
renderer.setSize(width, height); //设置three.js渲染区域的尺寸(像素px)
// 渲染器渲染方法
renderer.render(scene, camera); //执行渲染操作
// 渲染器Canvas画布属性
document.body.appendChild(renderer.domElement);
</script>

<template>
  <div>
  </div>
</template>

<style scoped>

</style>
