<template>
  <div class="hello">
  </div>
</template>
<script>
import * as THREE from 'three'
import * as Stats from 'stats.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { OBJLoader, MTLLoader } from 'three-obj-mtl-loader'

export default {
  name: 'scene',
  data() {
    return {
      car: null, //导入的汽车模型
      cube: null, //生成的方块 挥动
      camera: null, //相机
      renderer: null, //渲染器
      scene: null, //场景
      step: 0, //速度
      step1: 0, //cube速度
      stats: null,
      control: null,
      goahead: true //是否向前
    }
  },
  created() {
    this.init() //初始化，生成建筑物以及车辆
  },
  methods: {

    //随机生成范围数字
    RandomNumBoth(Min, Max) {
      var Range = Max - Min;
      var Rand = Math.random();
      var num = Min + Math.round(Rand * Range); //四舍五入
      return num;
    },
    move(fps, last) {
      /* fps刷新，目前不知道为啥不动）
        var now = new Date().getTime()
        var elapsed = now - last
        var fpsInterval = 1000 / fps
        if (elapsed > fpsInterval) {
          last = now - (elapsed % fpsInterval);
          //循环代码
          //cube.position.x = RandomNumBoth(-100, 100)
          //cube.position.y = RandomNumBoth(-100, 100)
          //蓝色是Z轴 红色是X轴 绿色是Y轴
          //car.position.z = step
          //cube.position.x = step
          //cube.position.z = Math.random()
          //*弹跳
          step += 0.04
          cube.position.x = (10 * (Math.cos(step)));
          cube.position.y = 2 + (5 * Math.abs(Math.sin(step)));
          //
          if (this.step > 20) this.goahead = false
          if (this.step < 0) this.goahead = true

          if (this.goahead) this.step += 0.05
          else this.step -= 0.05

          //this.cube.position.z = this.step
          if (this.car != null) this.car.position.z = this.step
          */
      /*弹跳
      this.step1 += 0.04
      this.cube.position.x = (10 * (Math.cos(this.step1)));
      this.cube.position.y = 2 + (5 * Math.abs(Math.sin(this.step1)));
      */

      if (this.step > 20) this.goahead = false
      if (this.step < 0) this.goahead = true

      if (this.goahead) this.step += 0.05
      else this.step -= 0.05

      this.cube.position.z = this.step
      if (this.car != null) this.car.position.z = this.step

    },
    //刷新函数
    animate() {

      requestAnimationFrame(this.animate) //刷新
      /*

      if (this.step > 20) this.goahead = false
      if (this.step < 0) this.goahead = true

      if (this.goahead) this.step += 0.05
      else this.step -= 0.05

      this.cube.position.z = this.step
      if (this.car != null) this.car.position.z = this.step
      */
      var last = new Date().getTime()

      //更新性能插件
      this.stats.update()

      //更新相机控件
      this.control.update()

      this.renderer.render(this.scene, this.camera);
      var lastnow = new Date().getTime()
      this.move(15, lastnow)

    },

    init() {
      //创建场景
      this.scene = new THREE.Scene()

      //创建相机 - 视野、长宽比、近、远裁剪平面
      // 10表示相机的远近，200表示切面的位置，200变成180拉动的时候容易看不全
      this.camera = new THREE.PerspectiveCamera(10, window.innerWidth / window.innerHeight, 0.1, 200)
      //let camera = new THREE.PerspectiveCamera(45, 1.25, 0.1, 200)
      this.camera.position.set(0, 0, 15)

      //创建渲染器 - 设置显示窗口大小
      this.renderer = new THREE.WebGLRenderer()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      //renderer.setSize(400, 303)
      document.body.appendChild(this.renderer.domElement)

      //创建坐标轴u
      var axes = new THREE.AxesHelper(30);
      //axes.size = ;
      axes.position.y = 1
      this.scene.add(axes);

      //创建方格平面
      let size = 150 //网格宽度
      let divisions = 50 //等分数
      let colorCenterLine = 0x444444 //中心线颜色
      let colorGrid = 0x88888 //网格线颜色
      let gridHelper = new THREE.GridHelper(size, divisions)
      //gridHelper.position.y = -2
      this.scene.add(gridHelper)


      // 创建平面
      var planeGeometry = new THREE.PlaneGeometry(30, 60);
      var planeMaterial = new THREE.MeshBasicMaterial({ color: 0xcccccc });
      var plane = new THREE.Mesh(planeGeometry, planeMaterial);
      plane.rotation.x = -0.5 * Math.PI;
      plane.position.x = 0;
      plane.position.y = -2.5;
      plane.position.z = 0;
      // 加入到场景 
      this.scene.add(plane);


      //创建车身
      //创建并设置大小
      var cubeGeometry = new THREE.BoxGeometry(1, 1, 3);

      //设置颜色线框显示否
      var cubeMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      this.cube = new THREE.Mesh(cubeGeometry, cubeMaterial);

      //设置cube的位置 
      this.cube.position.x = -3;
      this.cube.position.y = 0.5;
      this.cube.position.z = 1;
      this.cube.castShadow = true;

      //cube添加到场景中
      this.scene.add(this.cube);


      //创建性能检测
      this.stats = new Stats()
      this.stats.showPanel(0) // 0: fps, 1: ms, 2: mb, 3+: custom
      this.stats.width = 50 //性能不设置宽高会出现错误
      this.stats.height = 50
      document.body.appendChild(this.stats.dom)

      //创建相机控件
      this.control = new OrbitControls(this.camera, this.renderer.domElement)
      this.control.enableDamping = true
      this.control.dampingFactor = 0.25
      this.control.enableZoom = false

      this.importObj(this.scene, this.camera, this.renderer, this.stats, this.control)
      this.importGLTF(this.scene, this.camera, this.renderer, this.stats, this.control)

      console.log(this.scene.children.length)
      console.log(this.scene.children[0])
      console.log(this.scene.children[1])
      console.log(this.scene.children[2])
      console.log(this.scene.children[3])
      //this.importJSON(scene, camera, renderer, stats, control)
      // this.importJSON(scene, camera, renderer, stats, control)
    },

    //导入obj格式模型
    importObj(scene, camera, renderer, stats, control) {

      //设置相机位置
      camera.position.z = 130
      camera.position.y = 80

      var getFitScaleValue = function(obj) {
        var boxHelper = new THREE.BoxHelper(obj);
        boxHelper.geometry.computeBoundingBox();
        var box = boxHelper.geometry.boundingBox;
        var maxDiameter = Math.max((box.max.x - box.min.x), (box.max.y - box.min.y), (box.max.z - box.min.z));
        return this.camera.position.z / maxDiameter;
      }
      //设置模型到适合观察du的大小
      var setScaleToFitSize = function(obj) {
        var scaleValue = this.getFitScaleValue(obj);
        obj.scale.set(scaleValue, scaleValue, scaleValue);
        return scaleValue;
      }
      //导入汽车模型obj及渲染文件mtl
      var loader = new OBJLoader();
      var mtlloader = new MTLLoader();
      var _this = this
      mtlloader.load('/static/obj.mtl', function(materials) {
        materials.preload();
        loader.load('/static/obj.obj', function(group) {
          //var geometry = group.children[0].geometry;
          //geometry.attributes.uv2 = geometry.attributes.uv;
          //geometry.center();
          group.scale.set(0.0005, 0.0005, 0.0005) //设置车辆大小
          group.name = 'car'
          console.log(group)
          console.log(group.children[0])

          //将group改成mesh格式插入scene
          let carGeometry = new THREE.Geometry();
          for (let i = 0; i < group.children.length; i++) {
            let item = new THREE.Geometry().fromBufferGeometry(group.children[i].geometry)
            carGeometry.merge(item, group.children[i].matrix);
          }

          var carMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
          var mesh1 = new THREE.Mesh(carGeometry, carMaterial);
          mesh1.scale.set(0.0005, 0.0005, 0.0005)
          mesh1.position.x = 0
          mesh1.position.y = 0
          mesh1.position.z = 1
          //mesh.scale.multiplyScalar(25);
          //setScaleToFitSize(group)
          _this.car = mesh1
          scene.add(mesh1);
        }, xhr => {
          // called while loading is progressing
          console.log(`${( xhr.loaded / xhr.total * 100 )}% loaded`);
        }, error => {
          // called when loading has errors
          console.error('An error happened', error);
        })
      })

      var step = 0
      var goahead = true

      this.animate()
    },

    //导入GLTF格式模型
    importGLTF(scene, camera, renderer, stats, control) {
      //设置相机位置
      camera.position.z = 130
      camera.position.y = 80

      //创建GLTF加载器
      let loader = new GLTFLoader()

      // debugger
      loader.load('../static/dayawan1.gltf', gltf => {
        //缩放
        gltf.scene.scale.set(.1, .1, .1)

        //处理加载模型为黑色问题
        gltf.scene.traverse(child => {
          if (child.isMesh) {
            child.material.emissive = child.material.color
            child.material.emissiveMap = child.material.map
          }
        })

        scene.add(gltf.scene)
      }, xhr => {
        // called while loading is progressing
        console.log(`${( xhr.loaded / xhr.total * 100 )}% loaded`);
      }, error => {
        // called when loading has errors
        console.error('An error happened', error);
      })

      this.animate()
    },
    //导入JSON格式模型
    importJSON(scene, camera, renderer, stats, control) {
      //设置相机位置
      camera.position.z = 130
      camera.position.y = 80

      let loader = new THREE.ObjectLoader()

      loader.load('../static/obj.obj', group => {
        //处理加载模型为黑色问题
        group.traverse(child => {
          if (child.isMesh) {
            //赋值模型本身的颜色和纹理
            child.material.emissive = child.material.color
            child.material.emissiveMap = child.material.map
          }
        })

        scene.add(group)
      }, xhr => {
        // called while loading is progressing
        console.log(`${( xhr.loaded / xhr.total * 100 )}% loaded`);
      }, error => {
        // called when loading has errors
        console.error('An error happened', error);
      })

      this.animate()
    }
  },
  mounted() {
    this.animate()
  }
}

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#hello {}

</style>
