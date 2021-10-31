<template>
  <div
    ref="canvas"
    class="relative min-h-screen bg-gray-100 sm:pt-0"
  >

  </div>
</template>

<script>
  import * as THREE from 'three/build/three.module';

  export default {
    data() {
      return {
        camera: {},
        scene: {},
        renderer: {},
        geometry: {},
        material: {},
        mesh: {},
      };
    },
    mounted() {
      this.init();
    },
    methods: {
      animation(time) {
        this.mesh.rotation.x = time / 2000;
        this.mesh.rotation.y = time / 1000;

        this.renderer.render( this.scene, this.camera );
      },
      init() {
        this.camera = new THREE.PerspectiveCamera( 90, window.innerWidth / window.innerHeight, 0.2, 20 );
        this.camera.position.z = 1;

        this.scene = new THREE.Scene();

        this.geometry = new THREE.SphereGeometry( 0.2, 32, 16 );
        this.material = new THREE.MeshNormalMaterial();

        this.mesh = new THREE.Mesh( this.geometry, this.material );
        this.scene.add( this.mesh );

        this.renderer = new THREE.WebGLRenderer( { antialias: true, alpha: true } );
        this.renderer.setSize( window.innerWidth, window.innerHeight );
        this.renderer.setAnimationLoop( this.animation );
	      this.$refs.canvas.appendChild( this.renderer.domElement );
      },
    }
  }
</script>