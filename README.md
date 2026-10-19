
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>Highways Rush</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#142b22;color:white;font-family:Arial,sans-serif;text-align:center;overflow:hidden;touch-action:manipulation}
h2{margin:8px 0 3px;font-size:22px}
#info{display:flex;justify-content:space-around;max-width:420px;margin:6px auto;font-weight:bold}
#game{position:relative;width:min(100vw,420px);height:min(72vh,650px);min-height:350px;margin:auto;overflow:hidden;border:3px solid #d8e7d5;border-radius:10px;background:#79c9ed}
#sky{position:absolute;inset:0 0 62% 0;background:linear-gradient(#55b8ef,#c6ecff)}
.mountain{position:absolute;bottom:0;width:0;height:0;border-left:70px solid transparent;border-right:70px solid transparent;border-bottom:110px solid #738d9d}
.mountain:after{content:"";position:absolute;top:12px;left:-27px;border-left:27px solid transparent;border-right:27px solid transparent;border-bottom:43px solid white}
#field{position:absolute;inset:30% 0 0;background:linear-gradient(90deg,#348a3d,#83c84d,#368c3b)}
#pond{position:absolute;width:80px;height:40px;left:3%;top:44%;border-radius:50%;background:#49c8ef;border:4px solid #b9e7a2}
#road{position:absolute;top:0;bottom:0;left:18%;width:64%;background:#41464b;border-left:5px solid #fff2cf;border-right:5px solid #fff2cf;overflow:hidden}
.lane{position:absolute;top:-100%;height:300%;width:5px;background:repeating-linear-gradient(to bottom,#fff 0 35px,transparent 35px 70px);animation:roadmove .6s linear infinite}
.lane.one{left:33.33%}.lane.two{left:66.66%}
@keyframes roadmove{to{transform:translateY(70px)}}
.car{position:absolute;width:42px;height:72px;border-radius:12px 12px 8px 8px;border:3px solid #252525;box-shadow:inset 0 10px #ffffff50;z-index:5}
.car:before{content:"";position:absolute;left:6px;right:6px;top:12px;height:19px;border-radius:5px;background:#bceeff;border:2px solid #252525}
.car:after{content:"";position:absolute;left:5px;right:5px;bottom:7px;height:8px;border-radius:3px;background:#242a30}
#player{background:#e63232;bottom:18px;left:calc(50% - 21px)}
.enemy{background:#ffb329}
.tree{position:absolute;font-size:27px;z-index:2}
#overlay{position:absolute;inset:0;background:#0009;z-index:10;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:18px}
#overlay h1{font-size:30px;margin:8px}
#overlay p{font-size:15px}
button{border:0;border-radius:12px;padding:13px 24px;font-size:17px;font-weight:bold;background:#ffcc35;color:#17251c;cursor:pointer}
#controls{display:flex;justify-content:center;gap:35px;margin:10px auto}
.ctrl{width:100px;font-size:28px;padding:8px}
small{color:#d7e8dd}
</style>
</head>
<body>
<h2>🏁 HIGHWAYS RUSH 🏁</h2>
<div id="info">
<span>🏆 Score: <b id="score">0</b></span>
<span>🚗 Best: <b id="best">0</b></span>
</div>

<div id="game">
<div id="sky">
<div class="mountain" style="left:4%;border-bottom-width:105px"></div>
<div class="mountain" style="left:40%;border-bottom-width:135px;border-bottom-color:#91a7b3"></div>
<div class="mountain" style="left:72%;border-bottom-width:95px"></div>
</div>
<div id="field"></div>
<div id="pond"></div>
<div id="road">
<div class="lane one"></div><div class="lane two"></div>
</div>
<div class="tree" style="left:2%;top:32%">🌳</div>
<div class="tree" style="right:2%;top:38%">🌲</div>
<div class="tree" style="left:5%;top:65%">🌳</div>
<div class="tree" style="right:4%;top:70%">🌴</div>
<div id="player" class="car"></div>
<div id="overlay">
<h1>🏎️ HIGHWAYS RUSH</h1>
<p>Dodge enemy cars and drive as far as you can!</p>
<button id="start">START GAME ▶</button>
</div>
</div>

<div id="controls">
<button class="ctrl" id="left">⬅️</button>
<button class="ctrl" id="right">➡️</button>
</div>
<small>Use buttons or swipe to move your car</small>

<script>
const game=document.getElementById('game');
const player=document.getElementById('player');
const overlay=document.getElementById('overlay');
const scoreEl=document.getElementById('score');
const bestEl=document.getElementById('best');
const road=document.getElementById('road');

let lane=1,score=0,best=0,running=false;
let enemies=[],lastTime=0,spawnTimer=0,animationId=0;
let speed=180;

try{best=Number(localStorage.getItem('highwaysRushBest'))||0}catch(e){}
bestEl.textContent=best;

function carX(l){return road.offsetLeft+road.clientWidth*(l+.5)/3-21}
function placePlayer(){
 player.style.left=carX(lane)+'px';
}
function move(d){
 if(!running)return;
 lane=Math.max(0,Math.min(2,lane+d));
 placePlayer();
}
document.getElementById('left').addEventListener('pointerdown',()=>move(-1));
document.getElementById('right').addEventListener('pointerdown',()=>move(1));

let touchX=0;
game.addEventListener('touchstart',e=>{
 touchX=e.changedTouches[0].clientX;
},{passive:true});
game.addEventListener('touchend',e=>{
 let dx=e.changedTouches[0].clientX-touchX;
 if(Math.abs(dx)>25)move(dx>0?1:-1);
},{passive:true});

document.addEventListener('keydown',e=>{
 if(e.key==='ArrowLeft'||e.key==='a')move(-1);
 if(e.key==='ArrowRight'||e.key==='d')move(1);
});

function spawnEnemy(){
 const el=document.createElement('div');
 el.className='car enemy';
 let l=Math.floor(Math.random()*3);
 el.style.left=(road.offsetLeft+road.clientWidth*(l+.5)/3-21)+'px';
 el.style.top='-85px';
 game.appendChild(el);
 enemies.push({el,lane:l,y:-85});
}

function overlap(a,b){
 const x=a.getBoundingClientRect(),y=b.getBoundingClientRect();
 return x.left+7<y.right-7&&x.right-7>y.left+7&&
 x.top+8<y.bottom-8&&x.bottom-8>y.top+8;
}

function endGame(){
 running=false;
 cancelAnimationFrame(animationId);
 if(score>best){
  best=score;bestEl.textContent=best;
  try{localStorage.setItem('highwaysRushBest',best)}catch(e){}
 }
 overlay.innerHTML='<h1>🚘 GAME OVER</h1>'+
 '<p>Your score: '+score+'</p>'+
 '<button id="restart">PLAY AGAIN ↻</button>';
 overlay.style.display='flex';
 document.getElementById('restart').onclick=startGame;
}

function frame(t){
 if(!running)return;
 let dt=Math.min((t-lastTime)/1000,.04);
 lastTime=t;
 spawnTimer+=dt;
 if(spawnTimer>1.15){
  spawnTimer=0;
  spawnEnemy();
 }
 speed=180+Math.min(score*.7,100);
 for(let i=enemies.length-1;i>=0;i--){
  let en=enemies[i];
  en.y+=speed*dt;
  en.el.style.top=en.y+'px';
  if(overlap(player,en.el)){endGame();return}
  if(en.y>game.clientHeight){
   en.el.remove();enemies.splice(i,1);
   score++;scoreEl.textContent=score;
  }
 }
 animationId=requestAnimationFrame(frame);
}

function startGame(){
 cancelAnimationFrame(animationId);
 enemies.forEach(e=>e.el.remove());
 enemies=[];
 score=0;lane=1;speed=180;spawnTimer=0;
 scoreEl.textContent='0';
 overlay.style.display='none';
 placePlayer();
 running=true;
 lastTime=performance.now();
 animationId=requestAnimationFrame(frame);
}
document.getElementById('start').onclick=startGame;
window.addEventListener('resize',placePlayer);
placePlayer();
</script>
</body>
</html>
