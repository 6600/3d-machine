<template>
  <div class="app">
  </div>
</template>

<script>
import * as THREE from 'three/build/three.js'
import GLTFLoader from 'three-gltf-loader'
const OrbitControls = require('three-orbit-controls')(THREE)

export default {
  name: 'app',
  data () {
    return {
      mixer: null,
      camera: null,
      scene: null,
      clock: null,
      animations: null,
      mash: null,
      noErrorMaterial: null
    }
  },
  mounted () {
    const ws = new WebSocket("ws://xn--9tr.com:8005")
    ws.onopen = () => {
      console.log('连接建立成功!')
    }
    ws.onmessage = (mess) => {
      console.log('收到消息!')
      const data = JSON.parse(mess.data)
      if (data.state === true) {
        this.mash.material = new THREE.MeshLambertMaterial({
          emissive: 0xBA4A00,
          color: 0xBA4A00
        })
      } else {
        this.mash.material = this.noErrorMaterial
      }
    }
    this.clock = new THREE.Clock()
    this.init3D(this.$el, (Object3D) => {
      this.camera = Object3D.camera
      this.renderer = Object3D.renderer
      this.scene = Object3D.scene
      this.creatCube(Object3D.scene, Object3D.renderer, Object3D.camera)
    })
  },
  methods: {
    animate () {
      // console.log(this.clock.getDelta())
      requestAnimationFrame(this.animate)
      if (this.mixer !== null) this.mixer.update(this.clock.getDelta())
      if (this.controls) {
        this.controls.update()
      }
      this.renderScene()
    },
    renderScene () {
      this.renderer.render(this.scene, this.camera)
    },
    init3D (DOM, callBack) {
      const WIDTH = document.documentElement.clientWidth
      const HEIGHT = document.documentElement.clientHeight
      // 创建场景
      let scene = new THREE.Scene()
      // 配置渲染器
      let renderer = new THREE.WebGLRenderer({ antialias: true, preserveDrawingBuffer: true })
      // 设置渲染器大小
      renderer.setSize(WIDTH, HEIGHT)
      // 设置canvas背景色(clearColor)和背景色透明度（clearAlpha）
      renderer.setClearColor(0xffffff, 1)
      // 添加阴影
      renderer.shadowMap.enabled = true
      // 添加软阴影
      renderer.shadowMapSoft = true
      // 将渲染DOM追加到页面上
      DOM.appendChild(renderer.domElement)
      // 配置 透视 相机
      let camera = new THREE.PerspectiveCamera(45, WIDTH / HEIGHT, 1, 50000)
      // 设置相机位置
      camera.position.set(0, 10, 150)
      // 指定它看着原点方向
      camera.lookAt(scene.position)
      // 给场景追加相机
      scene.add(camera)

      // --------------------------- 创建光照 ---------------------------
      // 方向光源
      let object3d1 = new THREE.DirectionalLight('white', 0.8)
      object3d1.position.set(0, 0, 100)
      object3d1.name = 'Back light1'
      scene.add(object3d1)
      // 上方聚光灯
      let spotLight = new THREE.SpotLight( 0xffffff )
      spotLight.position.set(0, 60, 0)
      scene.add(spotLight)
      // 方向光源
      let object3d3 = new THREE.AmbientLight('white', 0.3)
      scene.add(object3d3)
      window.onresize = function () {
        const WIDTH = document.documentElement.clientWidth > 920 ? document.documentElement.clientWidth - 244 : 920 - 244
        const HEIGHT = document.documentElement.clientHeight > 576 ? document.documentElement.clientHeight - 218 : 576 - 218
        camera.aspect = WIDTH / HEIGHT
        camera.updateProjectionMatrix()
        renderer.setSize(WIDTH, HEIGHT)
      }
      callBack({scene, renderer, camera})
    },
    creatCube (scene, renderer, camera) {
      this.controls = new OrbitControls(camera, this.$el.childNodes[0])
      this.controls.maxPolarAngle = Math.PI * 0.5
      this.controls.autoRotate = true
      this.controls.minDistance = 10
      this.controls.maxDistance = 1000
      const loader = new GLTFLoader()
      loader.load('./test.gltf', (gltf) => {
        console.log(gltf)
        this.animations = gltf.animations
        if (this.animations && this.animations.length) {
          this.mixer = new THREE.AnimationMixer(gltf.scene)
          for (let i = 0; i < this.animations.length; i++) {
            let animation = this.animations[i]
            let action = this.mixer.clipAction(animation)
            action.play()
          }
        }
        gltf.scene.traverse( ( child ) => {
          if ( child.name === 'apolySurface2392' ) {
            this.mash = child
            this.noErrorMaterial = child.material
          }  
        })
        scene.add(gltf.scene)
      })
      this.animate()
    }
  }
}
</script>

<style lang="less">
.app, html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
button {
  position: fixed;
  right: 10px;
  bottom: 10px;
}
</style>
