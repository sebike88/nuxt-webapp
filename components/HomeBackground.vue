<template>
  <div
    ref="canvas"
    class="min-h-screen bg-gray-100 sm:pt-0 w-full relative"
  >

  </div>
</template>

<script>
  import * as THREE from 'three/build/three.module';
  import * as TWEEN from "@tweenjs/tween.js";
  import debounce from 'lodash.debounce';

  import { AdditiveBlending } from 'three';

  export default {
    data() {
      return {
        camera: {},
        scene: {},
        sceneCenter: {},
        renderer: {},
        geometry: {},
        particleGeometry: {},
        material: {},
        texture: {},
        mesh: {},
        spotLight: {},
        mouseX: 0,
        mouseY: 0,
        scrollPercent: 0,
        animationScripts: [],
      };
    },
    mounted() {
      this.init();
      this.addAnimationScripts();
      this.particles();
      window.addEventListener('resize', this.onWindowResize);
      document.body.addEventListener( 'pointermove', this.onPointerMove );
      document.addEventListener('scroll', debounce(this.scrollEvent, 500));
    },
    methods: {
      scrollEvent(event) {
        this.scrollPercent =
          (
            (document.documentElement.scrollTop || document.body.scrollTop) /
            ((document.documentElement.scrollHeight || document.body.scrollHeight) -
            document.documentElement.clientHeight)
          ) * 100;

        this.playScrollAnimations();

      },
      addAnimationScripts() {
        //  add an animation that moves the cube through first 40 percent of scroll
        this.animationScripts.push({
            start: 0,
            end: 10,
            func: () => {
              new TWEEN.Tween(this.camera)
              .to({
                position: {
                  x: 1,
                  y: 1,
                  z: 2,
                }
              }, 800)
              .easing(TWEEN.Easing.Cubic.Out)
              .onUpdate(() => {
                // this.camera.updateProjectionMatrix()
                // const percentage = this.lerp(0, 5, this.scalePercent(60, 80)) * 0.000001;
                // console.log(percentage)
                // this.camera.position.x = this.lerp(0, 5, this.scalePercent(60, 80))
                // this.camera.position.set(this.lerp(0, 5, this.scalePercent(60, 80)), 1, 2);
                // this.camera.position.set(coords.x, coords.y, 3);
                // this.camera.position.x = this.lerp(0, 5, this.scalePercent(60, 80))
              })
              .start();
            },
        });

        //  add an animation that rotates the cube between 40-60 percent of scroll
        this.animationScripts.push({
          start: 10,
          end: 30,
          func: () => {
            console.log('fire 2')
              // const coords = {
              //   x: this.camera.position.x,
              //   y: this.camera.position.y,
              //   z: this.camera.position.z,
              // };

              new TWEEN.Tween(this.camera)
              .to({
                position: {
                  x: 2,
                  y: 2,
                  z: 3,
                }
              }, 800)
              .easing( TWEEN.Easing.Quadratic.InOut )
              .onUpdate(() => {
                // this.camera.updateProjectionMatrix()
                // const percentage = this.lerp(0, 5, this.scalePercent(60, 80)) * 0.000001;
                // console.log(percentage)
                // this.camera.position.x = this.lerp(0, 5, this.scalePercent(60, 80))
                // this.camera.position.set(this.lerp(0, 5, this.scalePercent(60, 80)), 1, 2);
                // this.camera.position.set(coords.x, coords.y, 3);
                // this.camera.position.x = this.lerp(0, 5, this.scalePercent(60, 80))
              })
              .start();
          },
        });

        this.animationScripts.push({
          start: 60,
          end: 80,
          func: () => {
            this.camera.position.x = this.lerp(0, 5, this.scalePercent(60, 80))
          },
        });
      },
      playScrollAnimations() {
        this.animationScripts.forEach((item) => {
          if (this.scrollPercent >= item.start && this.scrollPercent < item.end) {
                item.func();
            }
        })
      },
      lerp(x, y, a) {
        return (1 - a) * x + a * y;
      },
      scalePercent(start, end) {
        return (this.scrollPercent - start) / (end - start);
      },
      animation(time) {
        requestAnimationFrame(this.animation);
        this.mesh.rotation.x = time / -10000;
        this.mesh.rotation.y = time / -50000;
        this.renderer.render( this.scene, this.camera );
				this.camera.lookAt( this.sceneCenter );
        TWEEN.update();
      },
      onPointerMove(event) {
				if ( event.isPrimary === false ) return;

				this.mouseX = event.clientX - (window.innerWidth / 2);
				this.mouseY = event.clientY - ( window.innerHeight / 2);

			},
      onWindowResize() {
        this.renderer.setSize( window.innerWidth, window.innerHeight );
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
      },
      particles() {
				const sprite = new THREE.TextureLoader().load( '/disc.png' );
				const vertices = [];

				for ( let i = 0; i < 70000; i ++ ) {

					const x = 100 * Math.random() - 50;
					const y = 100 * Math.random() - 50;
					const z = 100 * Math.random() - 50;
					vertices.push( x, y, z );

				}

        this.particleGeometry = new THREE.BufferGeometry();
				this.particleGeometry.setAttribute( 'position', new THREE.Float32BufferAttribute( vertices, 3 ) );

				const material = new THREE.PointsMaterial({
          size: 0.05,
          depthWrite: true,
          sizeAttenuation: true,
          blending: AdditiveBlending,
          color: 'rgb(255, 0, 0)',
          transparent: true,
          alphaMap: sprite,
        });
				material.color.setHSL( 1.0, 0.3, 0.7 );

				const particles = new THREE.Points( this.particleGeometry, material );
				this.scene.add( particles );
      },
      init() {
        this.camera = new THREE.PerspectiveCamera( 15, window.innerWidth / window.innerHeight, 0.1, 30 );
        this.camera.position.z = 2;
        this.scene = new THREE.Scene();
        this.geometry = new THREE.SphereGeometry( 0.2, 80, 80 );
        this.texture = new THREE.TextureLoader().load('/2k_mars.jpg')
        this.material = new THREE.MeshStandardMaterial({
          map: this.texture,
          bumpMap: this.texture,
          bumpScale: 0.005,
        });
        this.sceneCenter = new THREE.Vector3(0, 0, 0);
        this.spotLight = new THREE.SpotLight( 0xffffff );
        // this.spotLight.position.set( 100, 1500, 800 );
        this.spotLight.position.set( 0, 4, -3 );

        this.spotLight.castShadow = true;
        this.spotLight.shadow.mapSize.width = 1024;
        this.spotLight.shadow.mapSize.height = 1024;
        this.spotLight.shadow.camera.near = 500;
        this.spotLight.shadow.camera.far = 4000;
        this.spotLight.shadow.camera.fov = 30;

        this.scene.add(this.spotLight);

        this.mesh = new THREE.Mesh( this.geometry, this.material );
        this.mesh.rotation.y = 300;
        this.scene.add( this.mesh );

        this.renderer = new THREE.WebGLRenderer( {
          antialias: true,
          alpha: false,
        });
        this.renderer.toneMapping = THREE.CineonToneMapping;
        this.renderer.toneMappingExposure = 0.4;
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        this.renderer.setSize( window.innerWidth, window.innerHeight );
        // this.renderer.setAnimationLoop( this.animation );
        this.animation()
	      this.$refs.canvas.appendChild( this.renderer.domElement );
      },
    }
  }
</script>