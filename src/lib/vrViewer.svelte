<script lang="ts">
	import { onMount } from "svelte";
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import { VRButton } from 'three/addons/webxr/VRButton.js';
  import { XRControllerModelFactory } from 'three/examples/jsm/webxr/XRControllerModelFactory.js';
  import {
    PerspectiveCamera,
    Object3D,
    Clock,
    Quaternion
  } from "three";

  export let ifcModel: any
  let scene = new THREE.Scene();
  var camera: THREE.camera
  var renderer: THREE.renderer
  var controls: THREE.controls
  //Variables for VR hand controllers
  let controller1: THREE.controller, controller2: THREE.controller
  let controllerGrip1: THREE.Object3D, controllerGrip2: THREE.Object3D
  let cameraDolly: THREE.Object3D
  let dummyCam: THREE.camera


  onMount(() => {
    scene.add(ifcModel);
    // Prep Three Scene
    // lights
    const hemiLight = new THREE.HemisphereLight( 0xffffff, 0x000000, 1 );
    scene.add( hemiLight );
    const ambiLight = new THREE.AmbientLight( 0x404040 );
    scene.add( ambiLight );

    // renderer
    const container = document.getElementById("three-canvas");
    renderer = new THREE.WebGLRenderer({canvas: container, alpha: true });
    renderer.setPixelRatio( window.devicePixelRatio );
    renderer.setSize( window.innerWidth, window.innerHeight );
    renderer.xr.enabled = true;

    //Creates the camera (point of view of the user)
    camera = new PerspectiveCamera(75, window.innerWidth / window.innerHeight);
    camera.position.z = 15;
    camera.position.y = 13;
    camera.position.x = 8;

    // controls
    controls = new OrbitControls( camera, renderer.domElement );
    controls.minDistance = 1;
    controls.enableDamping = true;

    window.addEventListener( 'resize', onWindowResize );

    document.body.appendChild( VRButton.createButton( renderer ) );

    //VR Controllers 

    //Create a 3D object to carry the camera around XR session
    cameraDolly = new Object3D();
    cameraDolly.position.x = 0
    cameraDolly.position.y = 1.6
    cameraDolly.position.z = 5;
    cameraDolly.add(camera);
    scene.add(cameraDolly);

    //Add dummy camera to accurately get camera orientation in handleMovement function
    dummyCam = new Object3D();
    camera.add(dummyCam);

    controller1 = renderer.xr.getController( 0 );
    controller1.addEventListener( 'selectstart', allowMovement );
    controller1.addEventListener( 'selectend', stopMovement );
    scene.add( controller1 );

    controller2 = renderer.xr.getController( 1 );
    controller2.addEventListener( 'selectstart', allowMovement );
    controller2.addEventListener( 'selectend', stopMovement );
    scene.add( controller2 );

    const controllerModelFactory = new XRControllerModelFactory();

    controllerGrip1 = renderer.xr.getControllerGrip( 0 );
    controllerGrip1.add( controllerModelFactory.createControllerModel( controllerGrip1 ) );
    scene.add( controllerGrip1 );

    controllerGrip2 = renderer.xr.getControllerGrip( 1 );
    controllerGrip2.add( controllerModelFactory.createControllerModel( controllerGrip2 ) );
    scene.add( controllerGrip2 );

    // Needed to add controllers to dolly??
    cameraDolly.add(controller1);
    cameraDolly.add(controller2);
    cameraDolly.add(controllerGrip1);
    cameraDolly.add(controllerGrip2);

    animate()

  })

  function onWindowResize() {
    camera.updateProjectionMatrix();
    renderer.setSize( window.innerWidth, window.innerHeight );
  }

  const clock = new Clock();

  function animate() {
    requestAnimationFrame( animate );
    renderer.setAnimationLoop( render );
  }

  function render() {
    const dt = clock.getDelta();
    if (controller1 || controller2) { handleUserMovement(dt) }
    renderer.render( scene, camera );
  }

  //Functions to handle user movement around scene (3 of the 6 DoF)
  var letUserMove = false
  function allowMovement() { letUserMove = true }
  function stopMovement() { letUserMove = false }
  function handleUserMovement(dt) {
      if (letUserMove) {
          const speed = 2;
          const moveZ = -dt * speed
          const saveQuat = cameraDolly.quaternion.clone();
          var holder = new Quaternion()
          dummyCam.getWorldQuaternion(holder)
          cameraDolly.quaternion.copy(holder);
          cameraDolly.translateZ(moveZ);
          cameraDolly.quaternion.copy(saveQuat)
      }
  }

</script>

<canvas id="three-canvas"></canvas>

<style>
  #three-canvas{
      width: 100%;
      padding: 50px;
      height: 90%;
  }
</style>