<script lang="ts">
  import * as THREE from 'three';
  import { IFCLoader } from "three/examples/jsm/loaders/IFCLoader";
  import { GLTFExporter } from 'three/examples/jsm/exporters/GLTFExporter';
	import VrViewer from '$lib/vrViewer.svelte';
	import { onMount } from 'svelte';

  let files: FileList
  let arViewer = true
  let scale = 1
  let ifcModel: any
  let gltfModel: Blob
  let scene = new THREE.Scene();
  const ifcLoader = new IFCLoader();
  ifcLoader.ifcManager.setWasmPath( '/loaders/' );

  function switchViewModes() {
    arViewer = !arViewer
    // If coming from VR mode, we need to remove VR Button that was created.
    let vrButtonDomElement = document.getElementById("VRButton");
    if (vrButtonDomElement) {
      vrButtonDomElement.remove();
    }
  }

  function convertIfcModel() {
    if (files[0].name.split('.').pop() == "ifc") {
        let tempFile = URL.createObjectURL(files[0])
        ifcLoader.load( tempFile, function ( model: any ) {
            ifcModel = model;
            scene.clear(); // remove any previously loaded files
            scene.add( model );
            exportGLTF(scene);
        });
    } else {
        console.log("Selected file format is not compatible.  Please choose a .ifc file")
    } 
  }

  function exportGLTF( input: THREE.scene ) {
    const gltfExporter = new GLTFExporter();
    const options = {
      binary: true,
      maxTextureSize: 4096,
            forceIndices: true
    };
    gltfExporter.parse(input, function ( result: any ) {
      if ( result instanceof ArrayBuffer ) {
        saveArrayBuffer( result );
      } else {
        const output = JSON.stringify( result, null, 2 );
        saveString( output );
      }
      },
      function ( error: any ) {
        console.log( 'An error happened during parsing', error );
      },
      options
    );
  }

  function saveString( text: BlobPart ) {
        setPreview( new Blob( [ text ], { type: 'text/plain' } ) );
    }

    function saveArrayBuffer( buffer: ArrayBuffer ) {
        let blob = new Blob( [ buffer ], { type: 'application/octet-stream' } )
        setPreview( blob );
    }

    function setPreview( blob: Blob ) {
        gltfModel = blob
    }

</script>

<div class="view-container">
  <label for="fileInput">Choose IFC file to upload</label>
  <input type="file" id="fileInput" accept=".ifc" bind:files on:change="{convertIfcModel}"/>
  {#if ifcModel && gltfModel} <!-- Only load components when a model is available -->
    <button on:click={() => switchViewModes()}>Switch to {arViewer ? "VR" : "AR"} Viewer</button>
    {#if arViewer}
    <model-viewer id="gltfModel" class="arBlock" src={URL.createObjectURL(gltfModel)} scale="{scale} {scale} {scale}" ar ar-modes="webxr scene-viewer quick-look" seamless-poster shadow-intensity="1" camera-controls enable-pan></model-viewer>
    {:else}
    <VrViewer ifcModel={ifcModel}></VrViewer>  
    {/if}
  {/if}
 
  
</div>


<style>
  .view-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: white;
  }
  .arBlock {
    width: 100%;
    height: 90%;
    padding: 50px;
  }
</style>