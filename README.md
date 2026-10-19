
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Highways Rush</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#102d23;color:white;
 font-family:Arial,sans-serif;text-align:center;
 user-select:none;-webkit-user-select:none;
}
header{padding:18px 8px 12px}
h1{font-size:clamp(23px,6vw,38px);margin:0 0 20px}
.scoreboard{
 display:flex;justify-content:space-around;
 gap:8px;font-size:clamp(17px,4.5vw,25px);
 font-weight:bold;margin-bottom:15px;
}
#game{
 display:block;width:100%;height:min(69vh,850px);
 min-height:380px;border:6px solid #d5e8c5;
 border-radius:22px;background:#65c9ee;
 touch-action:pan-y;
}
.controls{
 display:flex;justify-content:center;gap:10%;
 padding:20px 12px 8px;
}
button{
 border:0;border-radius:23px;background:#ffcf32;
 color:#263238;font-size:42px;font-weight:bold;
 width:30%;max-width:205px;height:90px;
 box-shadow:0 5px 0 #d39b15;
 touch-action:manipulation;
}
button:active{transform:translateY(3px);box-shadow:0 2px 0 #d39b15}
.hint{font-size:17px;color:#d4e3d9;padding:8px 12px 25px}
#overlay{
 position:fixed;inset:0;background:#0009;
 display:flex;align-items:center;justify-content:center;
 padding:20px;z-index:5;
}
.panel{
 background:#102d23;border:3px solid #ffcf32;
 border-radius:24px;padding:25px 20px;
 width:min(90%,360px);box-shadow:0 10px 40px #0007;
}
.panel h2{font-size:32px;margin:5px 0 12px}
.panel p{font-size:18px;line-height:1.5}
.panel button{width:100%;max-width:none;height:65px;font-size:22px;margin-top:12px}
.hidden{display:none!important}
</style>
</head>
<body>

<header>
 <h1>🏁 HIGHWAYS RUSH 🏁</h1>
 <div class="scoreboard">
  <span>🏆 Score: <span id="score">0</span></span>
  <span>🚘 Best: <span id="best">0</span></span>
 </div>
</header>

<canvas id="game"></canvas>

<div class="controls">
 <button id="left" aria-label="Move left">⬅️</button>
 <button id="right" aria-label="Move right">➡️</button>
</div>
<div class="hint">Use arrows or swipe to move your car</div>

<div id="overlay">
 <div class="panel">
  <h2>🏁 HIGHWAYS RUSH</h2>
  <p>Dodge the traffic and make your best score!</p>
  <p>⬅️ ➡️ Move your car between lanes.</p>
  <button id="start">▶ START GAME</button>
 </div>
</div>

<script>
(() => {
"use strict";

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const scoreEl = document.getElementById("score");
const bestEl = document.getElementById("best");
const overlay = document.getElementById("overlay");
const startBtn = document.getElementById("start");

let W=360,H=600,dpr=1;
let running=false,gameOver=false;
let score=0,best=0,speed=230;
let playerLane=1,playerX=0;
let roadOffset=0,spawnTimer=0,lastTime=0;
let traffic=[];
let scenery=[];
let animationId=0;

try {
 best=Number(localStorage.getItem("highwaysRushBest"))||0;
} catch(e) {}
bestEl.textContent=best;

function resize(){
 const rect=canvas.getBoundingClientRect();
 W=Math.max(280,rect.width);
 H=Math.max(380,rect.height);
 dpr=Math.min(window.devicePixelRatio||1,2);
 canvas.width=Math.round(W*dpr);
 canvas.height=Math.round(H*dpr);
 ctx.setTransform(dpr,0,0,dpr,0,0);
 playerX=laneCenter(playerLane);
 draw();
}
window.addEventListener("resize",resize);

function roadWidth(){return W*0.62}
function roadLeft(){return (W-roadWidth())/2}
function laneWidth(){return roadWidth()/3}
function laneCenter(lane){return roadLeft()+laneWidth()*(lane+.5)}

function roundedRect(x,y,w,h,r,fill,stroke,line=3){
 ctx.beginPath();
 ctx.roundRect(x,y,w,h,r);
 ctx.fillStyle=fill;ctx.fill();
 if(stroke){ctx.lineWidth=line;ctx.strokeStyle=stroke;ctx.stroke();}
}

function drawBackground(){
 // Sky
 let sky=ctx.createLinearGradient(0,0,0,H*.5);
 sky.addColorStop(0,"#49b9f0");
 sky.addColorStop(1,"#d2f5ff");
 ctx.fillStyle=sky;ctx.fillRect(0,0,W,H);

 // Mountains
 ctx.fillStyle="#718b98";
 ctx.beginPath();
 ctx.moveTo(0,H*.48);ctx.lineTo(W*.2,H*.23);
 ctx.lineTo(W*.38,H*.46);ctx.lineTo(W*.63,H*.19);
 ctx.lineTo(W*.92,H*.48);ctx.lineTo(W,H*.42);
 ctx.lineTo(W,H*.6);ctx.lineTo(0,H*.6);ctx.fill();

 ctx.fillStyle="#f5f7ee";
 for(const m of [
  [W*.2,H*.23,W*.12,H*.16],
  [W*.63,H*.19,W*.15,H*.2]
 ]){
  ctx.beginPath();
  ctx.moveTo(m[0],m[1]+m[3]);
  ctx.lineTo(m[0]+m[2]*.5,m[1]);
  ctx.lineTo(m[0]+m[2],m[1]+m[3]);
  ctx.closePath();ctx.fill();
 }

 // Grass
 ctx.fillStyle="#389543";
 ctx.fillRect(0,H*.47,W,H*.53);

 // Pond
 ctx.fillStyle="#b5e48a";
 ctx.beginPath();ctx.ellipse(W*.11,H*.58,W*.09,H*.035,-.1,0,Math.PI*2);ctx.fill();
 ctx.fillStyle="#42c9ed";
 ctx.beginPath();ctx.ellipse(W*.11,H*.58,W*.078,H*.025,-.1,0,Math.PI*2);ctx.fill();

 // Moving roadside trees
 for(const s of scenery){
  const y=s.y;
  if(y < -30 || y > H+40) continue;
  const x=s.side===0?W*.075:W*.925;
  drawTree(x,y,s.type);
 }
}

function drawTree(x,y,type){
 ctx.save();ctx.translate(x,y);
 if(type===0){
  ctx.fillStyle="#76502e";ctx.fillRect(-3,2,6,18);
  ctx.fillStyle="#78c82e";
  ctx.beginPath();ctx.arc(0,-3,16,0,Math.PI*2);ctx.fill();
  ctx.fillStyle="#a4db39";
  ctx.beginPath();ctx.arc(-7,-7,8,0,Math.PI*2);ctx.fill();
 }else if(type===1){
  ctx.fillStyle="#14874c";
  for(let i=0;i<3;i++){
   ctx.beginPath();
   ctx.moveTo(0,-24+i*8);
   ctx.lineTo(-13,5+i*6);
   ctx.lineTo(13,5+i*6);
   ctx.closePath();ctx.fill();
  }
  ctx.fillStyle="#75502e";ctx.fillRect(-2,16,4,7);
 }else{
  ctx.fillStyle="#8b6039";ctx.fillRect(-3,0,6,20);
  ctx.strokeStyle="#16884b";ctx.lineWidth=4;
  ctx.beginPath();ctx.moveTo(0,5);ctx.lineTo(-13,-4);
  ctx.moveTo(0,9);ctx.lineTo(13,0);
  ctx.moveTo(0,0);ctx.lineTo(-8,-12);
  ctx.moveTo(0,1);ctx.lineTo(8,-14);ctx.stroke();
 }
 ctx.restore();
}

function drawRoad(){
 const left=roadLeft(),rw=roadWidth();
 ctx.fillStyle="#41464b";
 ctx.fillRect(left,0,rw,H);

 // Road edges
 ctx.fillStyle="#fff";
 ctx.fillRect(left-3,0,4,H);
 ctx.fillRect(left+rw-1,0,4,H);

 // Lane markings
 ctx.fillStyle="#f7f7f4";
 for(let lane=1;lane<3;lane++){
  const x=left+lane*laneWidth()-2;
  for(let y=-70+(roadOffset%100);y<H;y+=100){
   ctx.fillRect(x,y,4,65);
  }
 }
}

function drawCar(x,y,color,scale=1,isPlayer=false){
 ctx.save();
 ctx.translate(x,y);
 ctx.scale(scale,scale);

 const cw=47,ch=91;
 // Wheels
 ctx.fillStyle="#202328";
 roundedRect(-cw/2-3,-ch*.30,8,22,3,"#202328");
 roundedRect(cw/2-5,-ch*.30,8,22,3,"#202328");
 roundedRect(-cw/2-3,ch*.20,8,22,3,"#202328");
 roundedRect(cw/2-5,ch*.20,8,22,3,"#202328");

 // Body
 roundedRect(-cw/2,-ch/2,cw,ch,15,color,"#282a2c",4);

 // Windscreen
 roundedRect(-cw*.34,-ch*.30,cw*.68,ch*.30,8,
  isPlayer?"#a9ecff":"#b5edff","#343c40",3);

 // Front lights
 ctx.fillStyle="#fff1a6";
 ctx.fillRect(-cw*.34,-ch*.40,8,4);
 ctx.fillRect(cw*.17,-ch*.40,8,4);

 // Bumper
 roundedRect(-cw*.30,ch*.26,cw*.60,12,4,"#292d31",null);

 // Little highlights
 ctx.strokeStyle="#ffffff66";ctx.lineWidth=2;
 ctx.beginPath();ctx.moveTo(-cw*.29,-ch*.39);
 ctx.lineTo(-cw*.29,ch*.13);ctx.stroke();
 ctx.restore();
}

function draw(){
 if(!ctx)return;
 ctx.clearRect(0,0,W,H);
 drawBackground();
 drawRoad();

 for(const car of traffic){
  drawCar(car.x,car.y,car.color,car.scale,false);
 }

 drawCar(playerX,H*.79,"#f33f48",1,true);

 // Small road details
}

function resetGame(){
 score=0;speed=230;playerLane=1;
 playerX=laneCenter(1);
 roadOffset=0;spawnTimer=0;traffic=[];
 scenery=[];
 for(let i=0;i<14;i++){
  scenery.push({y:i*85-50,side:i%2,type:i%3});
 }
 scoreEl.textContent="0";
}

function spawnTraffic(){
 const lane=Math.floor(Math.random()*3);
 const colors=["#ffbd35","#29b8e8","#b06be5","#ff8847","#58d17d"];
 traffic.push({
  lane,
  x:laneCenter(lane),
  y:-65,
  color:colors[Math.floor(Math.random()*colors.length)],
  scale:.85+Math.random()*.1,
  passed:false
 });
}

function endGame(){
 if(!running)return;
 running=false;gameOver=true;
 if(score>best){
  best=score;
  bestEl.textContent=best;
  try{localStorage.setItem("highwaysRushBest",String(best));}catch(e){}
 }
 overlay.classList.remove("hidden");
 overlay.querySelector("h2").textContent="💥 GAME OVER!";
 overlay.querySelector("p").textContent="Your score: "+score+" 🏆";
 overlay.querySelectorAll("p")[1].textContent="Best score: "+best;
 startBtn.textContent="🔄 PLAY AGAIN";
}

function update(dt){
 if(!running)return;

 speed=230+Math.min(score*2,240);
 roadOffset=(roadOffset+speed*dt)%100;

 // Smooth lane movement
 const target=laneCenter(playerLane);
 playerX+=(target-playerX)*Math.min(1,dt*14);

 // Move trees
 for(const s of scenery){
  s.y+=speed*dt;
  if(s.y>H+25){
   s.y=-30-Math.random()*100;
   s.side=Math.random()<.5?0:1;
   s.type=Math.floor(Math.random()*3);
  }
 }

 // Spawn traffic
 spawnTimer+=dt;
 const interval=Math.max(.58,1.22-score*.006);
 if(spawnTimer>=interval){
  spawnTimer=0;
  spawnTraffic();
 }

 // Move traffic
 const py=H*.79;
 for(let i=traffic.length-1;i>=0;i--){
  const car=traffic[i];
  car.y+=speed*dt;
  car.x=laneCenter(car.lane);

  // Collision
  const dx=Math.abs(playerX-car.x);
  const dy=Math.abs(py-car.y);
  if(dx<laneWidth()*.53 && dy<72){
   endGame();return;
  }

  // Score for passing traffic
  if(!car.passed && car.y>py+65){
   car.passed=true;
   score++;
   scoreEl.textContent=score;
  }
  if(car.y>H+100)traffic.splice(i,1);
 }
}

function frame(now){
 const dt=Math.min((now-lastTime)/1000||0,.04);
 lastTime=now;
 update(dt);
 draw();
 animationId=requestAnimationFrame(frame);
}

function startGame(){
 cancelAnimationFrame(animationId);
 resetGame();
 gameOver=false;running=true;
 overlay.classList.add("hidden");
 lastTime=performance.now();
 animationId=requestAnimationFrame(frame);
}

function move(dir){
 if(!running)return;
 playerLane=Math.max(0,Math.min(2,playerLane+dir));
}

document.getElementById("left").addEventListener("pointerdown",e=>{
 e.preventDefault();move(-1);
});
document.getElementById("right").addEventListener("pointerdown",e=>{
 e.preventDefault();move(1);
});
startBtn.addEventListener("click",startGame);

let touchX=null,touchY=null;
canvas.addEventListener("touchstart",e=>{
 const t=e.changedTouches[0];
 touchX=t.clientX;touchY=t.clientY;
},{passive:true});

canvas.addEventListener("touchend",e=>{
 if(touchX===null)return;
 const t=e.changedTouches[0];
 const dx=t.clientX-touchX;
 const dy=t.clientY-touchY;
 if(Math.abs(dx)>25 && Math.abs(dx)>Math.abs(dy)){
  move(dx>0?1:-1);
 }
 touchX=null;touchY=null;
},{passive:true});

window.addEventListener("keydown",e=>{
 if(e.key==="ArrowLeft"||e.key==="a"||e.key==="A"){
  e.preventDefault();move(-1);
 }
 if(e.key==="ArrowRight"||e.key==="d"||e.key==="D"){
  e.preventDefault();move(1);
 }
 if((e.key==="Enter"||e.key===" ")&&!running){
  e.preventDefault();startGame();
 }
});

resize();
})();
</script>
</body>
</html>
