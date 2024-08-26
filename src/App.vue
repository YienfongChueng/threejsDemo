<script setup>
import {ref,nextTick} from 'vue'

// 引入three.js
import * as THREE from 'three';
// 引入轨道控制器扩展库OrbitControls.js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
// 引入dat.gui.js的一个类GUI
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';



// 虚拟场景 + 虚拟相机 -》渲染器（执行渲染操作）-》投影图（渲染结果）

// width和height用来设置Three.js输出的Canvas画布尺寸(像素px)
// const width = 800; //宽度
// const height = 500; //高度
const width = window.innerWidth; //窗口文档显示区的宽度作为画布宽度
const height = window.innerHeight; //窗口文档显示区的高度作为画布高度

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
console.log('查看材质对象',material);
// 模拟镜面反射，产生一个高光效果
// const material = new THREE.MeshPhongMaterial({
//     color: 0xff0000,
//     shininess: 20, //高光部分的亮度，默认30
//     specular: 0x444444, //高光部分的颜色
// });
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

////////// PointLightHelper：点光源辅助观察
const pointLightHelper = new THREE.PointLightHelper(pointLight, 10);
scene.add(pointLightHelper);
// 改变点光源位置，观察光照效果变化
// pointLight.position.set(0, 0, 0);
// 改变点光源位置，使用OrbitControls辅助观察
// pointLight.position.set(-400, -200, -300);

//环境光:没有特定方向，整体改变场景的光照明暗
const ambient = new THREE.AmbientLight(0xffffff, 0.4);
scene.add(ambient);

// 平行光
const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
// 设置光源的方向：通过光源position属性和目标指向对象的position属性计算
directionalLight.position.set(80, 100, 50);
// 方向光指向对象网格模型mesh，可以不设置，默认的位置是0,0,0
directionalLight.target = mesh;
scene.add(directionalLight);

// 对比不同入射角，mesh表面对光照的反射效果
// directionalLight.position.set(100, 0, 0);
// directionalLight.position.set(0, 100, 0);
// directionalLight.position.set(100, 100, 100);
// directionalLight.position.set(100, 60, 50);
//directionalLight.target默认指向坐标原点


//////////平行光辅助观察DirectionalLightHelper
// DirectionalLightHelper：可视化平行光
// const dirLightHelper = new THREE.DirectionalLightHelper(directionalLight, 5,0xff0000);
// scene.add(dirLightHelper);







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
// 渲染器锯齿属性.antialias
// 设备像素比window.devicePixelRatio
// 设置设备像素比.setPixelRatio()
// 设置背景颜色.setClearColor()
const renderer = new THREE.WebGLRenderer()
renderer.antialias = true
// 获取你屏幕对应的设备像素比.devicePixelRatio告诉threejs,以免渲染模糊问题
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setClearColor(0x444444, 1); //设置背景颜色

// 设置Canvas画布尺寸
renderer.setSize(width, height); //设置three.js渲染区域的尺寸(像素px)
// 渲染器渲染方法
// renderer.render(scene, camera); //执行渲染操作
// 渲染器Canvas画布属性
const webgl =ref()
nextTick(() => {
  webgl.value.appendChild(renderer.domElement);
})

// requestAnimationFrame实现周期性循环执行
// requestAnimationFrame默认每秒钟执行60次，但不一定能做到，要看代码的性能
// let i = 0;
// function render() {
//     i+=1;
//     console.log('执行次数'+i);
//     requestAnimationFrame(render);//请求再次执行函数render
// }
// render();
// 渲染函数
// function render() {
//     renderer.render(scene, camera); //执行渲染操作
//     mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
//     requestAnimationFrame(render);//请求再次执行渲染函数render，渲染下一帧
// }
// render();


//////////相机控件
// 旋转：拖动鼠标左键
// 缩放：滚动鼠标中键
// 平移：拖动鼠标右键

// 设置了渲染循环,相机控件OrbitControls就不用再通过事件change执行renderer.render(scene, camera);，
// 毕竟渲染循环一直在执行renderer.render(scene, camera);。

// 设置相机控件轨道控制器OrbitControls
// const controls = new OrbitControls(camera, renderer.domElement);
// // 如果OrbitControls改变了相机参数，重新调用渲染器渲染三维场景
// controls.addEventListener('change', function () {
//   // 浏览器控制台查看相机位置变化
//     // console.log('camera.position',camera.position);
//     renderer.render(scene, camera); //执行渲染操作
// });//监听鼠标、键盘事件


// 实例化一个gui对象
// const gui = new GUI();
//创建一个对象，对象属性的值可以被GUI库创建的交互界面改变
// const obj = {
//     x: 30,
// };
// 格式：.add(控制对象，对象具体属性，其他参数)
// gui增加交互界面，用来改变obj对应属性 
// gui.add(obj, 'x', 0, 100);

// const obj = {
//     x: 30,
//     y: 60,
//     z: 300,
// };
// gui界面上增加交互界面，改变obj对应属性
// gui.add(obj, 'x', 0, 100);
// gui.add(obj, 'y', 0, 50);
// gui.add(obj, 'z', 0, 60);

// 光照强度属性.intensity
// console.log('ambient.intensity',ambient.intensity);
// // 通过GUI改变mesh.position对象的xyz属性
// gui.add(ambient, 'intensity', 0, 2.0);

// gui.add(mesh.position, 'x', 0, 180);
// gui.add(mesh.position, 'y', 0, 180);
// gui.add(mesh.position, 'z', 0, 180);

// gui.add(ambient, 'intensity', 0, 2.0).name('环境光强度');
// gui.add(directionalLight, 'intensity', 0, 2.0).name('平行光强度');
// // 步长.step()方法可以设置交互界面每次改变属性值间隔是多少
// gui.add(ambient, 'intensity', 0, 2.0).name('环境光强度').step(0.1);

// const obj = {
//     x: 30,
//     color:0x00ffff,
//     scale: 0,
//     bool: false,
// };

// .addColor()生成颜色值改变的交互界面
// gui.addColor(obj, 'color').onChange(function(value){
//     mesh.material.color.set(value);
// });

// 参数3数据类型：数字，交互界面是一个鼠标可以拖动的拖动条，可以在一个区间改变属性的值   
// 当obj的x属性变化的时候，就把此时obj.x的值value赋值给mesh的x坐标
// gui.add(obj, 'x', 0, 180).onChange(function(value){
//     mesh.position.x = value;
// 	// 你可以写任何你想跟着obj.x同步变化的代码
// 	// 比如mesh.position.y = value;
// });
// 参数3数据类型：数组(下拉菜单)
// gui.add(obj, 'scale', [-100, 0, 100]).name('y坐标').onChange(function (value) {
//     mesh.position.y = value;
// });

// 参数3数据类型：对象(下拉菜单)
// gui.add(obj, 'scale', {
//     left: -100,
//     center: 0,
//     right: 100
//     // 左: -100,//可以用中文
//     // 中: 0,
//     // 右: 100
// }).name('位置选择').onChange(function (value) {
//     mesh.position.x = value;
// });

// 数据类型：布尔值
// 改变的obj属性数据类型是布尔值，交互界面是单选框
// gui.add(obj, 'bool').name('是否旋转');
// gui.add(obj, 'bool').onChange(function (value) {
//     // 点击单选框，控制台打印obj.bool变化
//     console.log('obj.bool',value);
// });

// gui.add(obj, 'bool').name('旋转动画');

// // 渲染循环
// function render() {
//     // 当gui界面设置obj.bool为true,mesh执行旋转动画
//     if (obj.bool) mesh.rotateY(0.01);
//     renderer.render(scene, camera);
//     requestAnimationFrame(render);
// }
// render();


const gui = new GUI(); //创建GUI对象 
const obj = {
     color: 0x00ffff,// 材质颜色
    bool: false,
};

// 渲染循环
function render() {
    // 当gui界面设置obj.bool为true,mesh执行旋转动画
    if (obj.bool) mesh.rotateY(0.01);
    renderer.render(scene, camera);
    requestAnimationFrame(render);
}
render();


nextTick(() => {
  // 创建材质子菜单
  // const matFolder = gui.addFolder('材质');
  // matFolder.close();
  // // 材质颜色color
  // matFolder.addColor(obj, 'color').onChange(function(value){
  //     material.color.set(value);
  // });
  // // 材质高光颜色specular
  // matFolder.addColor(obj, 'specular').onChange(function(value){
  //     material.specular.set(value);
  // });
  
  // 环境光子菜单
  const ambientFolder = gui.addFolder('环境光');
  // 环境光强度
  ambientFolder.add(ambient, 'intensity',0,2);
  
  // 平行光子菜单
  const dirFolder = gui.addFolder('平行光');
  dirFolder.close();//关闭菜单
  // 平行光强度
  dirFolder.add(directionalLight, 'intensity',0,2);
  const dirFolder2 = dirFolder.addFolder('位置');//子菜单的子菜单
  dirFolder2.close();//关闭菜单
  // 平行光位置
  dirFolder2.add(directionalLight.position, 'x',-400,400);
  dirFolder2.add(directionalLight.position, 'y',-400,400);
  dirFolder2.add(directionalLight.position, 'z',-400,400);
})










</script>

<template>
  <div ref="webgl" id="webgl"></div>
</template>

<style scoped>

</style>
