<template>
  <div class="hello"></div>
</template>

<script>

import * as THREE from 'three'

import {MTLLoader} from 'three/examples/jsm/loaders/MTLLoader.js'
import {OBJLoader} from 'three/examples/jsm/loaders/OBJLoader.js'
import config from '../libs/config.js'

const model = config.robot

let mouseX = 0
let mouseY = 0
let camera, scene, renderer
let windowHalfX = window.innerWidth / 2
let windowHalfY = window.innerHeight / 2
let that
const textureLoader = new THREE.TextureLoader()
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  mounted () {
    THREE.Material.side = THREE.DoubleSide
    that = this
    window.THREE = THREE
    this.init()
  },
  methods: {
    init: () => {
      if (scene != null) {
        return
      }
      const container = document.createElement('div')
      document.body.appendChild(container)
      scene = new THREE.Scene()

      camera = new THREE.PerspectiveCamera(model.fov, window.innerWidth / window.innerHeight, 2, 2000)
      camera.position.z = 250
      scene.add(new THREE.AmbientLight(0xcccccc, 0.4))
      camera.add(new THREE.PointLight(0xffffff, 0.8))
      scene.add(camera)
      // camera.position.set(100, 150, 100)
      renderer = new THREE.WebGLRenderer()
      renderer.outputEncoding = THREE.sRGBEncoding
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      container.appendChild(renderer.domElement)
      renderer.render(scene, camera)
      document.addEventListener('mousemove', that.onDocumentMouseMove)
      window.addEventListener('resize', that.onWindowResize)
      that.loadCube()
    },
    onWindowResize: () => {
      windowHalfX = window.innerWidth / 2
      windowHalfY = window.innerHeight / 2
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
    },

    onDocumentMouseMove: (event) => {
      mouseX = (event.clientX - windowHalfX) / 2
      mouseY = (event.clientY - windowHalfY) / 2
    },

    animate: () => {
      requestAnimationFrame(that.animate)
      that.render()
    },

    render: () => {
      camera.position.x = mouseX // += (-mouseX - camera.position.x) * 0.1
      camera.position.y = -mouseY // += (-mouseY - camera.position.y) * 0.1
      camera.lookAt(scene.position)
      renderer.render(scene, camera)
    },

    loadSquare: (that) => {
      const geometry = new THREE.BoxGeometry(2, 2, 2)
      const material = new THREE.MeshBasicMaterial({color: 0xff8716, wireframe: true})
      const cube = new THREE.Mesh(geometry, material)
      that.scene.add(cube)
    },
    loadCube: () => {
      new MTLLoader()
        .setPath(model.path)
        .load(`${model.name}.mtl`,
          (materials) => {
            materials.preload()
            new OBJLoader()
              .setMaterials(materials)
              .setPath(model.path)
              .load(`${model.name}.obj`,
                (obj) => {
                  console.log(scene, obj)
                  // 贴图
                  if (model.texture != null) {
                    obj.traverse(function (child) {
                      if (child instanceof THREE.Mesh) {
                        console.log(child.name)
                        for (let key in model.texture) {
                          if (child.name.indexOf(key) >= 0) {
                            child.material.opacity = 1
                            child.material.map = textureLoader.load(`${model.path}${model.texture[key]}`)
                          }
                        }
                      }
                    })
                  }

                  // 位置偏移
                  if (model.offsetY) {
                    obj.position.set(0, model.offsetY, 0)
                  }
                  scene.add(obj)
                  that.animate()
                },
                (xhr) => {
                  console.log('Progress obj : ', (xhr.loaded / xhr.total * 100) + '%')
                },
                (error) => {
                  console.error('3D模型加载失败', error)
                }
              )
          },
          (xhr) => {
            console.log('Progress mtl : ', (xhr.loaded / xhr.total * 100) + '%')
          },
          (error) => {
            console.error(error)
          })
    }
  }
}
</script>
