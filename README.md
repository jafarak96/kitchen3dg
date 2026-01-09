# kitchen3dg<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>كاونتر ألمنيوم 3D</title>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
<style>
  body{margin:0; overflow:hidden;}
  #ui{position:absolute; top:0; left:0; width:100%; background:#eee; padding:10px; z-index:10; font-family:Arial}
  input, select, button{margin:5px; font-size:16px;}
</style>
</head>
<body>

<div id="ui">
  <span>الطول (مم)</span><input id="length" value="3000">
  <span>العرض (مم)</span><input id="depth" value="600">
  <span>الارتفاع (مم)</span><input id="height" value="900">
  <span>الشكل</span>
  <select id="shape">
    <option value="STRAIGHT">مستقيم</option>
    <option value="L" selected>L</option>
    <option value="U">U</option>
  </select>
  <button onclick="updateKitchen()">تحديث</button>
</div>

<script>
let scene = new THREE.Scene();
let camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(4,4,3);

let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff, 0x444444, 1.2));

let root = new THREE.Group();
scene.add(root);

let doors = [];
let raycaster = new THREE.Raycaster();
let mouse = new THREE.Vector2();

// ==================== جسم الكاونتر ====================
function counterBody(w,d,h){
  const g=new THREE.BoxGeometry(w,d,h);
  const m=new THREE.MeshStandardMaterial({color:0xcccccc, metalness:0.9, roughness:0.3});
  const b=new THREE.Mesh(g,m);
  b.position.z=h/2;
  return b;
}

// ==================== أبواب ====================
function addDoors(parent,w,d,h){
  const unit=0.6;
  const doorH=h*0.85;
  const doorT=0.02;
  const mat=new THREE.MeshStandardMaterial({color:0x999999, metalness:0.8, roughness:0.4});
  const count=Math.floor(w/unit);
  for(let i=0;i<count;i++){
    const hinge=new THREE.Group();
    const geo=new THREE.BoxGeometry(unit-0.05,doorT,doorH);
    const door=new THREE.Mesh(geo,mat);
    door.position.x=(unit-0.05)/2;
    hinge.add(door);
    hinge.position.set(-w/2+i*unit,d/2+doorT/2,doorH/2);
    hinge.userData={open:false};
    doors.push(hinge);
    parent.add(hinge);
  }
}

// ==================== بناء الكاونتر ====================
function buildKitchen(type,l,d,h){
  doors=[];
  while(root.children.length) root.remove(root.children[0]);
  l/=1000; d/=1000; h/=1000;

  if(type==="STRAIGHT"){
    const g=new THREE.Group(); g.add(counterBody(l,d,h)); addDoors(g,l,d,h); root.add(g);
  }
  if(type==="L"){
    const a=new THREE.Group(); a.add(counterBody(l,d,h)); addDoors(a,l,d,h); root.add(a);
    const b=new THREE.Group(); b.rotation.z=Math.PI/2; b.position.x=l/2; b.position.y=-d/2;
    b.add(counterBody(d,d,h)); addDoors(b,d,d,h); root.add(b);
  }
  if(type==="U"){
    const left=new THREE.Group(); left.rotation.z=Math.PI/2; left.position.x=-l/2; left.position.y=-d/2;
    left.add(counterBody(d,d,h)); addDoors(left,d,d,h); root.add(left);
    const mid=new THREE.Group(); mid.add(counterBody(l,d,h)); addDoors(mid,l,d,h); root.add(mid);
    const right=new THREE.Group(); right.rotation.z=-Math.PI/2; right.position.x=l/2; right.position.y=-d/2;
    right.add(counterBody(d,d,h)); addDoors(right,d,d,h); root.add(right);
  }
}

// ==================== تحديث الكاونتر ====================
function updateKitchen(){
  const l=parseFloat(document.getElementById("length").value);
  const d=parseFloat(document.getElementById("depth").value);
  const h=parseFloat(document.getElementById("height").value);
  const type=document.getElementById("shape").value;
  buildKitchen(type,l,d,h);
}

// ==================== لمس الأبواب ====================
renderer.domElement.addEventListener('pointerdown',e=>{
  mouse.x=(e.clientX/window.innerWidth)*2-1;
  mouse.y=-(e.clientY/window.innerHeight)*2+1;
  raycaster.setFromCamera(mouse,camera);
  const hit=raycaster.intersectObjects(root.children,true);
  if(hit.length){
    const h=hit[0].object.parent;
    h.userData.open=!h.userData.open;
  }
});

// ==================== الحركة ====================
function animate(){
  requestAnimationFrame(animate);
  doors.forEach(h=>{const t=h.userData.open?-Math.PI/2:0; h.rotation.z+=(t-h.rotation.z)*0.1;});
  root.rotation.y+=0.002;
  renderer.render(scene,camera);
}
animate();

// إنشاء التصميم الافتراضي
updateKitchen();
</script>

</body>
</html>
