import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.161.0/build/three.module.js';
import { OrbitControls } from 'https://cdn.jsdelivr.net/npm/three@0.161.0/examples/jsm/controls/OrbitControls.js';
import { COLORS } from './math.js';

export function createViewer(container, callbacks) {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color('#edf3ef');
  scene.fog = new THREE.Fog('#edf3ef', 22, 65);
  const camera = new THREE.PerspectiveCamera(38, 1, 0.1, 200);
  camera.position.set(18, 16, 23);
  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
  renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.outputColorSpace = THREE.SRGBColorSpace;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.15;
  renderer.domElement.setAttribute('aria-label', 'Modelo 3D interactivo del edificio. Arrastra para rotar y usa la rueda para acercar.');
  container.prepend(renderer.domElement);

  scene.add(new THREE.HemisphereLight(0xffffff, 0x9aafa6, 2.1));
  const keyLight = new THREE.DirectionalLight(0xffffff, 3.2); keyLight.position.set(12, 22, 14); keyLight.castShadow = true; keyLight.shadow.mapSize.set(1024,1024); scene.add(keyLight);
  const fillLight = new THREE.DirectionalLight(0xc3e5d8, 1.3); fillLight.position.set(-14, 10, -8); scene.add(fillLight);

  const ground = new THREE.Mesh(new THREE.PlaneGeometry(80,80), new THREE.MeshStandardMaterial({color:0xe8efeb, roughness:1}));
  ground.rotation.x = -Math.PI/2; ground.position.y = -0.12; ground.receiveShadow = true; scene.add(ground);
  const grid = new THREE.GridHelper(40,40,0xb7cec4,0xd5e1db); grid.position.y = -0.1; scene.add(grid);
  const lot = new THREE.Mesh(new THREE.BoxGeometry(12,0.08,18), new THREE.MeshStandardMaterial({color:0xd9e6df, roughness:1}));
  lot.position.y = -0.03; lot.receiveShadow = true; scene.add(lot);
  const lotEdges = new THREE.LineSegments(new THREE.EdgesGeometry(new THREE.BoxGeometry(12,0.09,18)),new THREE.LineBasicMaterial({color:0x91b4a6})); lotEdges.position.y=-.02;scene.add(lotEdges);

  const building = new THREE.Group(); scene.add(building);
  const floorMeshes = [];
  let coreMesh = null, currentLevels = [], selected = -1, orthographic = false, orthoCamera = null, activeCamera = camera;
  const raycaster = new THREE.Raycaster(), pointer = new THREE.Vector2();
  let controls = new OrbitControls(activeCamera, renderer.domElement);
  controls.enableDamping = true; controls.dampingFactor = .07; controls.minDistance = 8; controls.maxDistance = 48; controls.maxPolarAngle = Math.PI * .48; controls.target.set(0,3,0);

  function resize() {
    const w = Math.max(1, container.clientWidth), h = Math.max(1, container.clientHeight);
    renderer.setSize(w,h,false);
    if (!orthographic) { camera.aspect = w/h; camera.updateProjectionMatrix(); }
    else { const frustum = 15; orthoCamera.left=-frustum*w/h/2;orthoCamera.right=frustum*w/h/2;orthoCamera.top=frustum/2;orthoCamera.bottom=-frustum/2;orthoCamera.updateProjectionMatrix(); }
  }
  const observer = new ResizeObserver(resize); observer.observe(container);

  function clearBuilding() {
    while (building.children.length) {
      const obj = building.children[0]; building.remove(obj);
      obj.traverse?.(child=>{ if(child.geometry) child.geometry.dispose(); if(child.material){ if(Array.isArray(child.material)) child.material.forEach(m=>m.dispose()); else child.material.dispose(); }});
    }
    floorMeshes.length = 0;
  }
  function rebuild(levels, lotWidth, lotLength, coreArea) {
    currentLevels = levels;
    clearBuilding();
    const sx = 8.4, sz = 12.6, floorScale = 0.42, heightScale = 0.75;
    const totalH = levels.reduce((s,l)=>s+l.height,0);
    let y = 0;
    levels.forEach((level,index)=>{
      const h = level.height*heightScale;
      const mat = new THREE.MeshStandardMaterial({color:COLORS[level.use]||0x55a58c, roughness:.42, metalness:.05, transparent:true, opacity:.78});
      const floor = new THREE.Mesh(new THREE.BoxGeometry(sx,h,sz),mat);
      floor.position.set(0,y+h/2,0); floor.castShadow=true;floor.receiveShadow=true;
      floor.userData.levelIndex=index; building.add(floor);floorMeshes.push(floor);
      const slab = new THREE.Mesh(new THREE.BoxGeometry(sx+.22,.11,sz+.22),new THREE.MeshStandardMaterial({color:0xf7faf8,roughness:.65}));
      slab.position.set(0,y+h+.03,0);slab.castShadow=true;slab.userData.levelIndex=index;building.add(slab);floorMeshes.push(slab);
      const edge = new THREE.LineSegments(new THREE.EdgesGeometry(new THREE.BoxGeometry(sx,h,sz)),new THREE.LineBasicMaterial({color:0x52776c,transparent:true,opacity:.7}));
      edge.position.copy(floor.position);edge.userData.levelIndex=index;building.add(edge);floorMeshes.push(edge);
      // Vertical glass-like facade strips give each floor a readable architectural rhythm.
      for(let j=-3;j<=3;j++){
        const mullion = new THREE.Mesh(new THREE.BoxGeometry(.035,h*.83,.035),new THREE.MeshStandardMaterial({color:0xf5faf7,roughness:.25,metalness:.2}));
        mullion.position.set(j*sx/7,y+h*.5,sz/2+.025);building.add(mullion);
      }
      y += h;
    });
    const roof = new THREE.Mesh(new THREE.BoxGeometry(sx+.38,.16,sz+.38),new THREE.MeshStandardMaterial({color:0x426a60,roughness:.6}));
    roof.position.y=y+.08;roof.castShadow=true;building.add(roof);
    const coreWidth = Math.max(1.1,Math.min(2.6,Math.sqrt(Math.max(1,coreArea))*0.12));
    coreMesh = new THREE.Mesh(new THREE.BoxGeometry(coreWidth,Math.max(.5,y),coreWidth*1.2),new THREE.MeshStandardMaterial({color:0xc5a66a,roughness:.55,transparent:true,opacity:.9}));
    coreMesh.position.set(sx*.22,Math.max(.5,y)/2,0);coreMesh.castShadow=true;building.add(coreMesh);
    // Stylized trees and site context.
    const treePositions=[[-6, -7],[6,-6],[-6,6],[6,7]];
    treePositions.forEach(([x,z])=>{
      const trunk=new THREE.Mesh(new THREE.CylinderGeometry(.09,.13,.8,6),new THREE.MeshStandardMaterial({color:0x927a58}));
      trunk.position.set(x,.4,z);building.add(trunk);
      const crown=new THREE.Mesh(new THREE.IcosahedronGeometry(.62,1),new THREE.MeshStandardMaterial({color:0x79a58a,roughness:1}));
      crown.position.set(x,1.15,z);crown.castShadow=true;building.add(crown);
    });
    // Selection outline
    updateSelection(selected);
    controls.target.y=Math.max(2,y*.48);
    resize();
  }
  function updateSelection(index) {
    selected=index;
    floorMeshes.forEach(mesh=>{
      if(mesh.userData.levelIndex===undefined || !mesh.material) return;
      const isSelected=mesh.userData.levelIndex===index;
      if(mesh.material.emissive){mesh.material.emissive.setHex(isSelected?0x1b8c7a:0x000000);mesh.material.emissiveIntensity=isSelected?.28:0;}
    });
  }
  function setGrid(visible){grid.visible=visible;}
  function setCameraMode(isOrtho) {
    if(isOrtho===orthographic) return;
    const pos=activeCamera.position.clone(), target=controls.target.clone();
    controls.dispose();
    orthographic=isOrtho;
    if(isOrtho){
      const w=Math.max(1,container.clientWidth),h=Math.max(1,container.clientHeight),f=15;
      orthoCamera=new THREE.OrthographicCamera(-f*w/h/2,f*w/h/2,f/2,-f/2,.1,200);orthoCamera.position.copy(pos);orthoCamera.lookAt(target);activeCamera=orthoCamera;
    } else {camera.position.copy(pos);camera.lookAt(target);activeCamera=camera;}
    controls=new OrbitControls(activeCamera,renderer.domElement);
    controls.enableDamping=true;controls.dampingFactor=.07;controls.minDistance=8;controls.maxDistance=48;controls.maxPolarAngle=Math.PI*.48;controls.target.copy(target);
    resize();
  }
  function resetView(){activeCamera.position.set(18,16,23);controls.target.set(0,3,0);activeCamera.lookAt(0,3,0);controls.update();}
  function onPointerMove(event){
    const rect=renderer.domElement.getBoundingClientRect();pointer.x=((event.clientX-rect.left)/rect.width)*2-1;pointer.y=-((event.clientY-rect.top)/rect.height)*2+1;
    raycaster.setFromCamera(pointer,activeCamera);
    const hits=raycaster.intersectObjects(floorMeshes,false).filter(h=>h.object.userData.levelIndex!==undefined);
    if(hits.length){const idx=hits[0].object.userData.levelIndex;callbacks.onHover?.(idx,event.clientX,event.clientY);renderer.domElement.style.cursor='pointer';}
    else {callbacks.onHover?.(-1);renderer.domElement.style.cursor='grab';}
  }
  function onClick(){raycaster.setFromCamera(pointer,activeCamera);const hits=raycaster.intersectObjects(floorMeshes,false).filter(h=>h.object.userData.levelIndex!==undefined);if(hits.length){callbacks.onSelect?.(hits[0].object.userData.levelIndex);}}
  renderer.domElement.addEventListener('pointermove',onPointerMove);renderer.domElement.addEventListener('click',onClick);
  let raf;
  function animate(){raf=requestAnimationFrame(animate);controls.update();renderer.render(scene,activeCamera);}
  animate();
  return {
    rebuild,updateSelection,setGrid,resetView,zoom(factor){activeCamera.position.sub(controls.target).multiplyScalar(factor).add(controls.target);controls.update();},
    setCameraMode,dispose(){cancelAnimationFrame(raf);observer.disconnect();renderer.domElement.removeEventListener('pointermove',onPointerMove);renderer.domElement.removeEventListener('click',onClick);controls.dispose();renderer.dispose();renderer.domElement.remove();}
  };
}
