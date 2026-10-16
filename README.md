<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Highway Rush 3D</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;overflow:hidden;background:#87ceeb;
  font-family:Arial,sans-serif;touch-action:none
}
canvas{display:block;width:100vw;height:100vh}
#hud{
 position:fixed;top:10px;left:10px;color:white;
 background:#0009;padding:12px;border-radius:12px;
 font-weight:bold;line-height:1.7;z-index:2
}
#controls{
 position:fixed;bottom:20px;width:100%;
 display:flex;justify-content:space-between;
 padding:0 22px;z-index:3
}
.group{display:flex;gap:12px}
button{
 border:2px solid white;border-radius:16px;
 background:#111b;color:white;font-size:23px;
 font-weight:bold;padding:15px 23px;
 touch-action:none
}
#message{
 position:fixed;inset:0;display:flex;
 align-items:center;justify-content:center;
 background:#0008;z-index:5;color:white;text-align:center
}
.panel{
 background:#101d2a;padding:28px;border-radius:20px;
 border:2px solid #fff8;max-width:340px;width:90%
}
.panel h1{color:#ffdf35}
#startBtn{background:#13a853;margin-top:15px}
#help{font-size:13px;color:#ddd}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="hud">
 🏆 Score: <span id="score">0</span><br>
 🛣️ Distance: <span id="distance">0</span> m<br>
 ⚡ Speed: <span id="speed">80</span> km/h
</div>

<div id="controls">
 <div class="group">
  <button id="left">⬅️</button>
  <button id="right">➡️</button>
 </div>
 <div class="group">
  <button id="slower">−</button>
  <button id="faster">+</button>
 </div>
</div>

<div id="message">
 <div class="panel">
  <h1 id="title">HIGHWAY RUSH 3D</h1>
  <p id="description">Race on the highway! Avoid traffic and enjoy the village scenery.</p>
  <p id="help">Cars • Bus • Train • Village • Speed Control</p>
  <button id="startBtn">▶ START GAME</button>
 </div>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let W,H;
function resize(){
 W=canvas.width=innerWidth*devicePixelRatio;
 H=canvas.height=innerHeight*devicePixelRatio;
 canvas.style.width=innerWidth+"px";
 canvas.style.height=innerHeight+"px";
 ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
 W=innerWidth;H=innerHeight;
}
addEventListener("resize",resize);resize();

let running=false,gameOver=false;
let playerX=0,speed=80,score=0,distance=0;
let roadOffset=0,traffic=[],scenery=[],trainOffset=0;
let lastTime=0,spawnTimer=0;
let keys={left:false,right:false};

function roadWidth(){return Math.min(W*.78,540)}
function horizon(){return H*.31}
function roadAt(y){
 const t=Math.max(0,Math.min(1,(y-horizon())/(H-horizon())));
 return W/2+(W*.5-roadWidth()*.5)*0;
}
function perspective(y){
 return Math.max(.05,(y-horizon())/(H-horizon()));
}
function laneX(lane,y){
 const p=perspective(y);
 return W/2+lane*roadWidth()*.34*p;
}

function polygon(points,color){
 ctx.fillStyle=color;ctx.beginPath();
 points.forEach((p,i)=>i?ctx.lineTo(p[0],p[1]):ctx.moveTo(p[0],p[1]));
 ctx.closePath();ctx.fill();
}

function drawBackground(){
 // Sky
 let sky=ctx.createLinearGradient(0,0,0,H);
 sky.addColorStop(0,"#42a5f5");
 sky.addColorStop(1,"#d9f5ff");
 ctx.fillStyle=sky;ctx.fillRect(0,0,W,H);

 // Sun
 ctx.fillStyle="#fff3a0";
 ctx.beginPath();ctx.arc(W*.82,H*.12,Math.min(W,H)*.055,0,Math.PI*2);ctx.fill();

 // Distant hills
 polygon([[0,horizon()+35],[W*.12,horizon()-20],
 [W*.28,horizon()+30],[W*.47,horizon()-35],
 [W*.7,horizon()+25],[W*.88,horizon()-15],
 [W,horizon()+20],[W,H],[0,H]],"#75b957");

 // Village fields
 ctx.fillStyle="#65ad43";
 ctx.fillRect(0,horizon(),W,H-horizon());

 // Village houses on both sides
 for(let i=0;i<9;i++){
   let y=horizon()+((i*97+roadOffset*.5)%(H-horizon()+50));
   let p=perspective(y);
   if(y>H)continue;
   let size=12+p*36;
   drawHouse(W*.08+(i%2)*W*.13,y,size,p);
   drawHouse(W*.92-(i%2)*W*.13,y,size,p);
 }

 // Road
 const rw=roadWidth();
 polygon([[W/2-rw*.06,horizon()],[W/2+rw*.06,horizon()],
 [W/2+rw*.72,H],[W/2-rw*.72,H]],"#383d43");

 // Road shoulders
 polygon([[W/2-rw*.065,horizon()],[W/2-rw*.055,horizon()],
 [W/2-rw*.67,H],[W/2-rw*.74,H]],"#e5e5e5");
 polygon([[W/2+rw*.055,horizon()],[W/2+rw*.065,horizon()],
 [W/2+rw*.74,H],[W/2+rw*.67,H]],"#e5e5e5");

 // Road edges
 ctx.strokeStyle="#fff";ctx.lineWidth=3;
 ctx.beginPath();ctx.moveTo(W/2-rw*.06,horizon());ctx.lineTo(W/2-rw*.72,H);ctx.stroke();
 ctx.beginPath();ctx.moveTo(W/2+rw*.06,horizon());ctx.lineTo(W/2+rw*.72,H);ctx.stroke();

 // Lane markings
 for(let i=0;i<16;i++){
  let y=horizon()+((i*65+roadOffset*1.8)%(H-horizon()));
  let p=perspective(y),dash=5+p*24;
  ctx.fillStyle="#fff4bd";
  for(let lane of [-.33,.33]){
   let x=W/2+lane*rw*p;
   ctx.fillRect(x-2*p,y,4*p,dash);
  }
 }

 // Railway tracks on right side
 let railX=W*.91;
 ctx.strokeStyle="#66594d";
 ctx.lineWidth=5;
 ctx.beginPath();
 ctx.moveTo(railX-14,horizon());
 ctx.lineTo(railX-14,H);
 ctx.moveTo(railX+14,horizon());
 ctx.lineTo(railX+14,H);
 ctx.stroke();
 for(let i=0;i<24;i++){
  let y=horizon()+((i*35+roadOffset*1.3)%(H-horizon()));
  let p=perspective(y);
  ctx.strokeStyle="#c2b6a0";ctx.lineWidth=2+p*3;
  ctx.beginPath();ctx.moveTo(railX-22*p,y);ctx.lineTo(railX+22*p,y);ctx.stroke();
 }
}

function drawHouse(x,y,size,p){
 if(y>H+size)return;
 ctx.fillStyle="#d9b58a";
 ctx.fillRect(x-size*.5,y-size*.65,size,size*.65);
 polygon([[x-size*.65,y-size*.65],[x,y-size*1.1],[x+size*.65,y-size*.65]],"#9e4031");
 ctx.fillStyle="#5d3c2b";
 ctx.fillRect(x-size*.12,y-size*.34,size*.24,size*.34);
 ctx.fillStyle="#9ee7ff";
 ctx.fillRect(x-size*.38,y-size*.52,size*.19,size*.17);
 ctx.fillRect(x+size*.19,y-size*.52,size*.19,size*.17);
}

function drawCar(x,y,w,h,color,isPlayer=false){
 // Wheels
 ctx.fillStyle="#171717";
 ctx.fillRect(x-w*.52,y-h*.68,w*.18,h*.27);
 ctx.fillRect(x+w*.34,y-h*.68,w*.18,h*.27);
 ctx.fillRect(x-w*.52,y-h*.18,w*.18,h*.25);
 ctx.fillRect(x+w*.34,y-h*.18,w*.18,h*.25);

 // Body
 polygon([[x-w*.4,y],[x-w*.48,y-h*.23],
 [x-w*.35,y-h*.72],[x-w*.22,y-h*.94],
 [x+w*.22,y-h*.94],[x+w*.35,y-h*.72],
 [x+w*.48,y-h*.23],[x+w*.4,y]],color);

 // Windows
 polygon([[x-w*.28,y-h*.57],[x-w*.18,y-h*.83],
 [x+w*.18,y-h*.83],[x+w*.28,y-h*.57]],"#aee8ff");
 ctx.fillStyle="#fff";
 ctx.fillRect(x-w*.32,y-h*.12,w*.16,h*.07);
 ctx.fillRect(x+w*.16,y-h*.12,w*.16,h*.07);
 if(isPlayer){
  ctx.fillStyle="#ff3333";
  ctx.fillRect(x-w*.34,y-h*.02,w*.18,h*.06);
  ctx.fillRect(x+w*.16,y-h*.02,w*.18,h*.06);
 }
}

function drawBus(x,y,w,h){
 ctx.fillStyle="#f5c928";
 ctx.fillRect(x-w/2,y-h,w,h);
 ctx.fillStyle="#222";
 for(let i=0;i<4;i++){
  ctx.fillStyle="#a9e7ff";
  ctx.fillRect(x-w*.4+i*w*.205,y-h*.88,w*.16,h*.22);
 }
 ctx.fillStyle="#333";
 ctx.fillRect(x-w*.44,y-h*.18,w*.2,h*.18);
 ctx.fillRect(x+w*.24,y-h*.18,w*.2,h*.18);
 ctx.fillStyle="#e33";
 ctx.fillRect(x-w*.42,y-h*.07,w*.12,h*.04);
 ctx.fillRect(x+w*.3,y-h*.07,w*.12,h*.04);
}

function drawTrain(){
 // A train moves alongside the road
 const x=W*.91;
 const span=H-horizon();
 const y=horizon()+((trainOffset)%span);
 const count=3;
 for(let i=0;i<count;i++){
  let yy=y-i*105;
  if(yy<horizon()-100)yy+=span+count*105;
  let p=perspective(Math.max(horizon(),yy));
  let w=30+p*24,h=55+p*45;
  ctx.fillStyle=i===0?"#e4f0f4":"#d6e0e8";
  ctx.fillRect(x-w/2,yy-h,w,h);
  ctx.fillStyle="#237bb4";
  ctx.fillRect(x-w/2,yy-h*.38,w,h*.18);
  ctx.fillStyle="#172c40";
  for(let j=0;j<3;j++){
   ctx.fillRect(x-w*.38+j*w*.29,yy-h*.8,w*.19,h*.16);
  }
  ctx.fillStyle="#111";
  ctx.fillRect(x-w*.42,yy-h*.12,w*.2,h*.12);
  ctx.fillRect(x+w*.22,yy-h*.12,w*.2,h*.12);
 }
}

function drawTraffic(){
 traffic.forEach(o=>{
  let p=perspective(o.y);
  let x=laneX(o.lane,o.y);
  let w=Math.max(15,roadWidth()*.105*p);
  let h=w*1.65;
  if(o.type==="bus")drawBus(x,o.y,w*1.35,h*1.5);
  else drawCar(x,o.y,w,h,o.color);
 });
}

function drawPlayer(){
 let x=W/2+playerX*roadWidth()*.34;
 let w=Math.min(68,W*.14);
 let h=w*1.65;
 drawCar(x,H*.86,w,h,"#e62626",true);
}

function reset(){
 playerX=0;speed=80;score=0;distance=0;
 roadOffset=0;trainOffset=0;traffic=[];
 spawnTimer=0;gameOver=false;
 document.getElementById("score").textContent="0";
 document.getElementById("distance").textContent="0";
 document.getElementById("speed").textContent="80";
}

function endGame(){
 running=false;gameOver=true;
 document.getElementById("title").textContent="GAME OVER!";
 document.getElementById("description").innerHTML=
 "🏆 Score: "+score+"<br>Distance: "+Math.floor(distance)+" m";
 document.getElementById("startBtn").textContent="🔄 RESTART";
 document.getElementById("message").style.display="flex";
}

function update(dt){
 if(!running)return;
 roadOffset+=speed*dt*.7;
 trainOffset+=speed*dt*.45;
 distance+=speed*dt/3.6;
 score=Math.floor(distance/5);

 if(keys.left)playerX-=dt*1.8;
 if(keys.right)playerX+=dt*1.8;
 playerX=Math.max(-.9,Math.min(.9,playerX));

 spawnTimer+=dt;
 if(spawnTimer>1.1){
  spawnTimer=0;
  let lane=[-1,0,1][Math.floor(Math.random()*3)];
  traffic.push({
   lane:lane,y:horizon()+10,
   type:Math.random()<.22?"bus":"car",
   color:["#1687e8","#ffb52e","#9b65dd","#f1f1f1"][Math.floor(Math.random()*4)]
  });
 }

 traffic.forEach(o=>o.y+=speed*dt*.65);
 traffic=traffic.filter(o=>o.y<H+100);

 const px=W/2+playerX*roadWidth()*.34;
 const py=H*.86;
 for(let o of traffic){
  if(o.y>py-65&&o.y<py+25){
   const ox=laneX(o.lane,o.y);
   if(Math.abs(px-ox)<roadWidth()*.105){
    endGame();return;
   }
  }
 }

 document.getElementById("score").textContent=score;
 document.getElementById("distance").textContent=Math.floor(distance);
 document.getElementById("speed").textContent=speed;
}

function render(){
 ctx.clearRect(0,0,W,H);
 drawBackground();
 drawTrain();
 drawTraffic();
 drawPlayer();
}

function loop(t){
 let dt=Math.min((t-lastTime)/1000||0,.05);
 lastTime=t;
 update(dt);render();
 requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

document.getElementById("startBtn").addEventListener("click",()=>{
 reset();
 running=true;
 document.getElementById("message").style.display="none";
});

function hold(id,key){
 const el=document.getElementById(id);
 el.addEventListener("pointerdown",e=>{
  e.preventDefault();keys[key]=true;
 });
 ["pointerup","pointercancel","pointerleave"].forEach(ev=>
  el.addEventListener(ev,()=>keys[key]=false)
 );
}
hold("left","left");
hold("right","right");

document.getElementById("faster").addEventListener("pointerdown",()=>{
 speed=Math.min(180,speed+10);
});
document.getElementById("slower").addEventListener("pointerdown",()=>{
 speed=Math.max(30,speed-10);
});

addEventListener("keydown",e=>{
 if(e.key==="ArrowLeft"||e.key==="a")keys.left=true;
 if(e.key==="ArrowRight"||e.key==="d")keys.right=true;
 if(e.key==="ArrowUp")speed=Math.min(180,speed+10);
 if(e.key==="ArrowDown")speed=Math.max(30,speed-10);
});
addEventListener("keyup",e=>{
 if(e.key==="ArrowLeft"||e.key==="a")keys.left=false;
 if(e.key==="ArrowRight"||e.key==="d")keys.right=false;
});
</script>
</body>
</html> 
