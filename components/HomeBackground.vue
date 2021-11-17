<template>
  <div
    ref="canvas"
    class="home-background min-h-screen bg-gray-100 sm:pt-0 w-full fixed"
  >
  <div class="home-background__bg absolute w-full h-full"></div>
  </div>
</template>

<script>
  import * as THREE from 'three/build/three.module';
  import * as TWEEN from "@tweenjs/tween.js";
  // import debounce from 'lodash.debounce';

  import { AdditiveBlending } from 'three';

  export default {
    data() {
      return {
        camera: {},
        scene: {},
        sceneCenter: {},
        renderer: {},
        geometry: {},
        gridGeometry: {},
        particleGeometry: {},
        material: {},
        orbitControls: {},
        texture: {},
        mesh: {},
        spotLight: {},
        mouseX: 0,
        mouseY: 0,
        scrollPercent: 0,
        animationScripts: [],
      };
    },
    created() {
      this.$nuxt.$on('slide-change', (index) => {
        this.playScrollAnimations(index);
      });
    },
    mounted() {
      this.init();
      this.addAnimationScripts();
      this.particles();
      window.addEventListener('resize', this.onWindowResize);
      document.body.addEventListener( 'pointermove', this.onPointerMove );
      document.addEventListener('scroll', this.scrollEvent);
    },
    methods: {
      scrollEvent(event) {
        this.scrollPercent =
          (
            (document.documentElement.scrollTop || document.body.scrollTop) /
            ((document.documentElement.scrollHeight || document.body.scrollHeight) -
            document.documentElement.clientHeight)
          ) * 100;
      },
      addAnimationScripts() {
        //  add an animation that moves the cube through first 40 percent of scroll
        this.animationScripts.push({
            start: 0,
            end: 5,
            func: () => {
              new TWEEN.Tween(this.sceneCenter)
              .to({
                x: 0,
                y: 0,
                z: 0,
              }, 1500)
              .easing( TWEEN.Easing.Quartic.InOut ).start();

              new TWEEN.Tween(this.camera)
              .to({
                position: {
                  x: 0,
                  y: 0,
                  z: 1.5,
                }
              }, 1500)
              .easing( TWEEN.Easing.Quartic.InOut )
              .onUpdate(() => {})
              .onComplete(() => {}).start();
            },
        });

        //  add an animation that rotates the cube between 40-60 percent of scroll
        this.animationScripts.push({
          start: 5,
          end: 50,
          func: () => {
            new TWEEN.Tween(this.sceneCenter)
            .to({
              x: 0.3,
              y: 0,
              z: 0,
            }, 1500)
            .easing( TWEEN.Easing.Quartic.InOut ).start();

            new TWEEN.Tween(this.camera)
            .to({
              position: {
                x: 1,
                y: 0,
                z: this.lerp(1, -20, this.scalePercent(60, 80) * 0.05),
              }
            }, 1500)
            .onStart(() => {})
            .easing( TWEEN.Easing.Quartic.InOut )
            .onUpdate(() => {})
            .start();
          },
        });

        this.animationScripts.push({
          start: 5,
          end: 50,
          func: () => {
            new TWEEN.Tween(this.sceneCenter)
            .to({
              x: 0,
              y: 0.3,
              z: 0,
            }, 1500)
            .easing( TWEEN.Easing.Quartic.InOut ).start();

            new TWEEN.Tween(this.camera)
            .to({
              position: {
                x: 1.2,
                y: 0,
                z: 1.5,
              }
            }, 1500)
            .onStart(() => {})
            .easing( TWEEN.Easing.Quartic.InOut )
            .onUpdate(() => {})
            .start();
          },
        });
      },
      playScrollAnimations(index) {
        this.animationScripts[index].func();
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
        this.orbitControls.update()
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

				for ( let i = 0; i < 90000; i++ ) {

					const x = 80 * Math.random() - 50;
					const y = 80 * Math.random() - 50;
					const z = 80 * Math.random() - 50;
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
      setGridGeometry() {
        const faceColorMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff, vertexColors: THREE.FaceColors } );
        const sphereGeometry = new THREE.SphereGeometry(0.25,  24, 24);
        for ( let i = 0; i < sphereGeometry.faces.length; i++ ) 
	      {
          const face = sphereGeometry.faces[ i ];	
          face.color.setRGB( 0, 0, 0.8 * Math.random() + 0.2 );
        }
        const sphere = new THREE.Mesh( sphereGeometry, faceColorMaterial );
        sphere.position.set(0, 0, 0);
        this.scene.add(sphere);
      },
      async setOrbitControls() {
        const {OrbitControls} = await import('three/examples/jsm/controls/OrbitControls')
        this.orbitControls = new OrbitControls(this.camera, this.renderer.domElement)
        this.orbitControls.autoRotate = true
        this.orbitControls.autoRotateSpeed = 1;
      },
      async init() {
        this.camera = new THREE.PerspectiveCamera( 15, window.innerWidth / window.innerHeight, 0.1, 30 );
        this.camera.position.z = 1.5;
        this.scene = new THREE.Scene();
        this.geometry = new THREE.SphereGeometry( 0.2, 80, 80 );
        this.texture = new THREE.TextureLoader().load('/2k_mars.jpg')
        this.textureBump = new THREE.TextureLoader().load('/marsbump1k.jpg')
        this.material = new THREE.MeshStandardMaterial({
          map: this.texture,
          bumpMap: this.texture,
          bumpScale: 0.005,
        });
        this.sceneCenter = new THREE.Vector3(0, 0, 0);
        this.spotLight = new THREE.SpotLight( 0xffffff );
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
        this.renderer.toneMapping = THREE.ACESFilmicToneMapping;
        this.renderer.toneMappingExposure = 0.6;
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        this.renderer.setSize( window.innerWidth, window.innerHeight );
        await this.setOrbitControls()
        this.animation();
	      this.$refs.canvas.appendChild( this.renderer.domElement );
      },
    }
  }
</script>

<style lang="scss">
@import "~/assets/scss/components/HomeBackground.scss";
</style>