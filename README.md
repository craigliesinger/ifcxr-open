# ifcXR - Open Source

This project is the open source version of ifcXR (ifcxr.com).  ifcXR utilizes open source projects Three.js, IFC.js, and ModelViewer.dev.  This repo shows how these three projects can combine to create an XR (AR + VR) viewer for the AEC industry which uses IFC files.

ifcxr.com combines these technologies with a storage and sharing solution using Firebase as a Platform.  If you want to quickly and easily use this technology you can signup at ifcXR.  If you want to build your own solution - all the key steps are found in this project.

## Key Elements
Static folder contains a folder 'loaders', which contains web-ifc.wasm.  This is needed for ifcjs loader to work.  The .wasm loader must match appropriate version of ifcjs loader in three.js version.

ModelViewer package is loaded in index header (src/app.html)
```html
    <!-- Google Model Viewer Package needed for AR Viewer-->
		<script type="module" src="https://unpkg.com/@google/model-viewer/dist/model-viewer.min.js"></script>
```
This project unitilizes one single page (src/routes/+page.svelte) (side note, check out Svelte framework for understanding folder structure and file namings... and Svelte is awesome).  This file provides logic for loading IFC file and converting to .gltf (for AR viewer).

Logic for VR Viewer is in a component at src/lib/vrViewer.svelte as more code is needed to properly setup Three for VR viewing and its cleaner to have in its own component.

As of now, VR Controlers only have functionality to move user.  Upon trigger selection user will move towards direction they are looking.

ifcxr.com project only saves .gltf files upon upload, so movement around model is most keen there - but as seen in this open source project, the IFC file can be used in the VR viewer allowing for additional functionality such as item selection for detail / visibility.  See IFC.js project for examples.


## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of this app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
