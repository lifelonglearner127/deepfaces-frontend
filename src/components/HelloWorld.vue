<template>
  <div>
    <button @click="onCamera(0)">Camera 1</button>
    <button @click="onCamera(2)">Camera 2</button>
    <button @click="onCamera(3)">Camera 3</button>
    <div id="container"></div>
  </div>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data: function() {
    return {
      renderer: null,
      scene: null,
      camera: null,
      controls: null,
      isCamera0On: false,
      isCamera1On: false,
      isCamera2On: false
    }
  },
  mounted () {
    let container = document.getElementById( 'container' );

    this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
    this.camera.position.set(1, 2, -3)
    this.camera.lookAt(0, 1, 0)

    this.scene = new THREE.Scene()
    this.scene.background = new THREE.Color(0xa0a0a0)
    this.scene.fog = new THREE.Fog(0xa0a0a0, 10, 50)

    let hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444)
    hemiLight.position.set(0, 20, 0)
    this.scene.add(hemiLight)

    let dirLight = new THREE.DirectionalLight(0xffffff)
    dirLight.position.set(-3, 10, -10)
    dirLight.castShadow = true
    dirLight.shadow.camera.top = 2
    dirLight.shadow.camera.bottom = - 2
    dirLight.shadow.camera.left = - 2
    dirLight.shadow.camera.right = 2
    dirLight.shadow.camera.near = 0.1
    dirLight.shadow.camera.far = 40
    this.scene.add(dirLight)

    var mesh = new THREE.Mesh(
      new THREE.PlaneBufferGeometry( 100, 100 ),
      new THREE.MeshPhongMaterial( { color: 0x999999, depthWrite: false } )
    )
    mesh.rotation.x = - Math.PI / 2
    mesh.receiveShadow = true
    this.scene.add( mesh )

    let loader = new GLTFLoader()
    let vm = this
		loader.load('building.glb', function (gltf) {
      let model = gltf.scene
      model.scale.set(0.02, 0.02, 0.02)
			vm.scene.add(model)
      model.traverse( function ( object ) {

        if ( object.isMesh ) {
          object.castShadow = true
        }
      })
      vm.animate()
    } )

    this.renderer = new THREE.WebGLRenderer( { antialias: true } )
    this.renderer.setPixelRatio( window.devicePixelRatio )
    this.renderer.setSize( window.innerWidth, window.innerHeight )
    this.renderer.outputEncoding = THREE.sRGBEncoding
    this.renderer.shadowMap.enabled = true
    container.appendChild( this.renderer.domElement )

    this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    this.controls.addEventListener( 'change', this.render ); // use if there is no animation loop
    this.controls.minDistance = 2;
    this.controls.maxDistance = 10;
    this.controls.target.set( 0, 0, - 0.2 );
    this.controls.update();

  },
  methods: {
    animate() {
      requestAnimationFrame(this.animate)
      this.renderer.render( this.scene, this.camera )
    },
    render () {
      this.renderer.render( this.scene, this.camera )
    },
    onCamera (cameraIndex) {
      let cameraOn = false
      switch(cameraIndex) {
        case 0:
          this.isCamera0On = !this.isCamera0On
          cameraOn = this.isCamera0On
          break
        case 2:
          this.isCamera1On = !this.isCamera1On
          cameraOn = this.isCamera1On
          break
        case 3:
          this.isCamera2On = !this.isCamera2On
          cameraOn = this.isCamera2On
          break
      }
      let cameraMesh = "mesh_" + cameraIndex
      this.scene.traverse(function(object) {
        if (object.isMesh && object.name === cameraMesh) {
          if (cameraOn) {
            object.material.color.setHex(0xff0000)
          } else {
            object.material.color.setHex(0xffffff )
          }
        }
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
