<script>
import { ref, onMounted } from "vue";
import * as THREE from "three"; //Three.js core
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

export default {
  setup() {
    let myCanvas = ref(null);

    function init() {
      /*---------------------------------------
        Canvas Element
      ----------------------------------------*/
      const canvas = {
        element: myCanvas,
        width: myCanvas.value.offsetWidth,
        height: myCanvas.value.offsetWidth,
        ratio: myCanvas.value.offsetWidth / myCanvas.value.offsetWidth,
      };

      /*---------------------------------------
        Colors
      ----------------------------------------*/
      const colors = {
        dark: new THREE.Color(0x444444),
        grey: new THREE.Color(0xecf0f1),
        light: new THREE.Color(0xdddddd),
        white: new THREE.Color(0xffffffff),
        red: new THREE.Color(0xff0000),
        green: new THREE.Color(0x00ff00),
        blue: new THREE.Color(0x0000ff),
        sun: new THREE.Color(0xffeeb1),
        black: new THREE.Color(0x080820),
        cian: new THREE.Color(0x049ef4),
      };

      /*---------------------------------------
        Scene
      ----------------------------------------*/
      const scene = new THREE.Scene();

      /*---------------------------------------
        Camera
      ----------------------------------------*/
      let fov = 100;
      let aspect = canvas.ratio;
      let near = 0.1;
      let far = 1000;

      const perspectiveCamera = new THREE.PerspectiveCamera(
        fov,
        aspect,
        near,
        far
      );
      perspectiveCamera.position.set(3, 3, 3);

      /*---------------------------------------
        Material
      ----------------------------------------*/
      let standardMaterial = {
        color: colors.green,
        wireframe: false,
        metalness: 0.5,
        roughness: 0.1,
      };
      const newStandardMaterial = new THREE.MeshStandardMaterial(standardMaterial);

      let phongMaterial = {
        color: colors.red,
        wireframe: false,
      };
      const newPhongMaterial = new THREE.MeshPhongMaterial(phongMaterial);

      /*---------------------------------------
        Geometrys
      ----------------------------------------*/
      let width = 3;
      let height = 0.1;
      let depth = 3;

      let radius = 1;
      let widthSegments = 100;
      let heightSegments = 100;
      let detail = 0;

      const sphere = new THREE.SphereGeometry(
        radius,
        widthSegments,
        heightSegments
      );
      const sphereGeometry = new THREE.Mesh(sphere, newPhongMaterial);
      sphereGeometry.position.set(0, 1, 0);
      sphereGeometry.castShadow = true;
      sphereGeometry.receiveShadow = true;

      const box = new THREE.BoxGeometry(width, height, depth);
      const boxGeometry = new THREE.Mesh(box, newStandardMaterial);
      boxGeometry.castShadow = true;
      boxGeometry.receiveShadow = true;

      const icosahedron = new THREE.IcosahedronGeometry(radius, detail);
      const isoconGeometry = new THREE.Mesh(icosahedron, newStandardMaterial);

      //Call this function to add load file into scene
      loadGLTFFile(scene)

      /*---------------------------------------
        Lights
      ----------------------------------------*/
      let helperColor = colors.red;
      let intensity = 1;
      let distance = 1;
      let decay = 1;
      let dummy = 0.1;

      //Point Light
      const pointLight = new THREE.PointLight(
        colors.red,
        intensity,
        distance,
        decay
      );

      pointLight.position.set(0, 3, 0);
      pointLight.castShadow = true;
      const pointHelper = new THREE.PointLightHelper(
        pointLight,
        dummy,
        helperColor
      );

      //Directional Light
      const directionalLight = new THREE.DirectionalLight(
        colors.white,
        intensity
      );
      directionalLight.castShadow = true;
      directionalLight.position.set(3, 3, 3);
      directionalLight.target.position.set(0, 0, 0);

      const directionalHelper = new THREE.DirectionalLightHelper(
        directionalLight,
        dummy,
        helperColor
      );

      //Hemisfer Light
      const hemisferLight = new THREE.HemisphereLight(
        colors.white,
        colors.black,
        intensity
      );

      /*---------------------------------------
        Grid Helper
      ----------------------------------------*/
      let gridSize = 4;
      let gridDivisions = 20;
      const gridHelper = new THREE.GridHelper(gridSize, gridDivisions);

      /*---------------------------------------
      Axis Helper
      ----------------------------------------*/
      const axisHelper = new THREE.AxesHelper(2);
      let AxisX = colors.red;
      let AxisY = colors.blue;
      let AxisZ = colors.green;
      axisHelper.setColors(AxisX, AxisY, AxisZ);

      /*---------------------------------------
        Scene Additions
      ----------------------------------------*/
      scene.add(perspectiveCamera);

      //Add geometries in scene
      //scene.add(boxGeometry);
      //scene.add(sphereGeometry)
      //scene.add(isoconGeometry);

      //Add lights in scene
      //scene.add(pointLight);
      scene.add(hemisferLight);
      scene.add(directionalLight);
      scene.add(directionalLight.target);

      //Add helpers in scene
      //scene.add(pointHelper);
      scene.add(directionalHelper);
      scene.add(axisHelper);
      //scene.add(gridHelper);

      /*---------------------------------------
        Render Setup
      ----------------------------------------*/
      const renderer = new THREE.WebGLRenderer({
        canvas: canvas.element.value,
        alpha: true,
        antialias: true,
      });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(canvas.width, canvas.height);
      renderer.shadowMap.enabled = true;
      renderer.outputEncoding = THREE.sRGBEncoding;

      /*---------------------------------------
        Controls
      ----------------------------------------*/
      const controls = new OrbitControls(
        perspectiveCamera,
        renderer.domElement
      );
      controls.enableDamping = false;

      renderIntoDOM();
      console.log("Rendering...");

      function renderIntoDOM() {
        //Render Animation
        isoconGeometry.rotation.x += 0;
        isoconGeometry.rotation.y += 0.02;
        isoconGeometry.rotation.z += 0;

        // Performing Render Instances
        renderer.render(scene, perspectiveCamera);

        // Recursive call of render function
        window.requestAnimationFrame(renderIntoDOM);
      }
    }

    /*---------------------------------------
      Loader GLTF file
    ----------------------------------------*/
    //Put the 3D model inside of public/models folder to be load.
    //Change the path '/models/scene.gltf' inside the function to get the best reference

    function loadGLTFFile(scene) {
      const loaderGLTF = new GLTFLoader();
      loaderGLTF.load(
        "/models/scene.gltf",
        (gltf) => {
          let model = gltf.scene;
          //model.position.set(0, 1, 0);
          scene.add(model);
        },
        undefined,
        function (error) {
          console.error(error);
        }
      );
    }

    // Mounted Lifecycle Hooks
    onMounted(() => {
      init();
    });

    // Expose the states to the template
    return {
      myCanvas,
    };
  },
};
</script>
<template>
  <div class="canvas-container" id="container">
    <canvas id="canvas" ref="myCanvas" class="canvas-element">
      <!--The geometry will be rendered here-->
    </canvas>
  </div>
</template>

<style scoped>
.canvas-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
canvas {
  width: 100%;
  height: 100%;
}
</style>