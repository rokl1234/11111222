<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no">
<title>3D迷宫闯关</title>

<style>
body { margin:0; overflow:hidden; background:#87ceeb; font-family:sans-serif; }
#ui{
  position:fixed;
  bottom:20px;
  left:50%;
  transform:translateX(-50%);
  display:grid;
  grid-template-columns:70px 70px 70px;
  grid-template-rows:70px 70px 70px;
  gap:10px;
}
button{
  font-size:24px;
  border:none;
  border-radius:15px;
  background:#ffffffcc;
}
#level{
  position:fixed;
  top:10px;
  left:10px;
  background:#fff;
  padding:6px 12px;
  border-radius:10px;
}
</style>
</head>

<body>
<div id="level">关卡 1</div>

<div id="ui">
  <div></div><button onclick="move(0,-1)">⬆️</button><div></div>
  <button onclick="move(-1,0)">⬅️</button><div></div><button onclick="move(1,0)">➡️</button>
  <div></div><button onclick="move(0,1)">⬇️</button><div></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152/build/three.min.js"></script>
<script>
let scene, camera, renderer;
let maze = [];
let size = 7;
let level = 1;
const cell = 4;

let player, goal;
let mazeObjects = [];

init();
nextLevel();
animate();

function init(){
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb);

  camera = new THREE.PerspectiveCamera(60, innerWidth/innerHeight, 0.1, 1000);

  renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(innerWidth, innerHeight);
  document.body.appendChild(renderer.domElement);

  scene.add(new THREE.AmbientLight(0xffffff,0.5));
  const light = new THREE.DirectionalLight(0xffffff,1);
  light.position.set(10,20,10);
  scene.add(light);

  window.addEventListener("resize",()=>{
    camera.aspect = innerWidth/innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);
  });
}

/* ===== 迷宫生成 ===== */
function generateMaze(n){
  maze = Array.from({length:n},()=>Array(n).fill(1));

  function carve(x,y){
    maze[y][x]=0;
    const dirs=[[2,0],[-2,0],[0,2],[0,-2]].sort(()=>Math.random()-0.5);
    for(const [dx,dy] of dirs){
      const nx=x+dx, ny=y+dy;
      if(nx>0 && nx<n-1 && ny>0 && ny<n-1 && maze[ny][nx]){
        maze[y+dy/2][x+dx/2]=0;
        carve(nx,ny);
      }
    }
  }
  carve(1,1);
}

/* ===== 清理关卡 ===== */
function clearMaze(){
  mazeObjects.forEach(o=>scene.remove(o));
  mazeObjects=[];
}

/* ===== 创建关卡 ===== */
function nextLevel(){
  clearMaze();
  size = 7 + level*2;
  generateMaze(size);

  const offset = (size*cell)/2 - cell/2;

  const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(size*cell, size*cell),
    new THREE.MeshLambertMaterial({color:0xdddddd})
  );
  floor.rotation.x = -Math.PI/2;
  scene.add(floor);
  mazeObjects.push(floor);

  for(let y=0;y<size;y++){
    for(let x=0;x<size;x++){
      if(maze[y][x]){
        const wall = new THREE.Mesh(
          new THREE.BoxGeometry(cell,3,cell),
          new THREE.MeshLambertMaterial({color:0xffffff})
        );
        wall.position.set(x*cell-offset,1.5,y*cell-offset);
        scene.add(wall);
        mazeObjects.push(wall);
      }
    }
  }

  player = new THREE.Mesh(
    new THREE.BoxGeometry(1.5,2.5,1.5),
    new THREE.MeshLambertMaterial({color:0x4aa3ff})
  );
  player.position.set(cell-offset,1.25,cell-offset);
  scene.add(player);
  mazeObjects.push(player);

  goal = new THREE.Mesh(
    new THREE.BoxGeometry(2,0.5,2),
    new THREE.MeshLambertMaterial({color:0x44ff44})
  );
  goal.position.set((size-2)*cell-offset,0.25,(size-2)*cell-offset);
  scene.add(goal);
  mazeObjects.push(goal);

  document.getElementById("level").innerText = "关卡 " + level;
}

/* ===== 移动 ===== */
function move(dx,dy){
  const px = Math.round(player.position.x/cell);
  const pz = Math.round(player.position.z/cell);

  const nx = px + dx;
  const nz = pz + dy;

  if(maze[nz] && maze[nz][nx]===0){
    player.position.x = nx*cell;
    player.position.z = nz*cell;
  }

  if(player.position.distanceTo(goal.position) < 1.5){
    level++;
    nextLevel();
  }
}

/* ===== 渲染 ===== */
function animate(){
  requestAnimationFrame(animate);

  camera.position.set(
    player.position.x - 8,
    8,
    player.position.z - 8
  );
  camera.lookAt(player.position);

  renderer.render(scene,camera);
}
</script>
</body>
</html>
