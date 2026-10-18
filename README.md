<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Arrow Maze</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#111;color:white;font-family:Arial;text-align:center}
h1{font-size:28px;margin:22px 0 8px}
p{color:#ddd;font-size:15px}
#wrap{width:min(94vw,600px);margin:15px auto;background:white}
svg{display:block;width:100%;height:auto;touch-action:manipulation}
.wall{fill:none;stroke:#050505;stroke-width:7;stroke-linecap:round;stroke-linejoin:round}
.arr{fill:#050505}
#trail{fill:none;stroke:#ed1c2e;stroke-width:5;stroke-linecap:round;stroke-linejoin:round}
#dot{fill:#ed1c2e;stroke:white;stroke-width:1.5}
button{padding:13px 22px;margin:6px;border:0;border-radius:10px;background:#ed1c2e;color:white;font-weight:bold;font-size:16px}
#msg{min-height:24px;font-weight:bold}
#controls{display:grid;grid-template-columns:repeat(3,60px);gap:7px;justify-content:center;margin:12px}
#controls button{margin:0;padding:12px 0;font-size:23px}
</style>
</head>
<body>
<h1>ARROW MAZE</h1>
<p>Raasta chuno aur laal player ko exit tak le jao.</p>
<div id="wrap">
<svg id="board" viewBox="0 0 400 400">
<rect width="400" height="400" fill="#fff"/>
<g class="wall">
<path d="M25 25H125V85H75V115H125V145H165V25H205V115"/>
<path d="M25 25V135 M55 50V145H105V175H165"/>
<path d="M225 25H375V155H315V190H375"/>
<path d="M225 75H265V95H285V50H345V85"/>
<path d="M225 105H245V145H220V190H190V165H175"/>
<path d="M25 165H75 M25 165V230H105"/>
<path d="M55 190H150V230H190V255H235"/>
<path d="M375 190V250H335V290H375V375H205V325H175V300H145V345H115V300H85V375H25V260H65V285H45V305"/>
<path d="M25 260H90V285H65"/>
<path d="M105 375V315H85 M125 260V300H150"/>
<path d="M175 300V270H205V300H225V325"/>
<path d="M235 255H270V280H300V310H335"/>
<path d="M250 190H285V230H310"/>
<path d="M265 230V270H285V325H310"/>
<path d="M235 325H250V350H280"/>
<path d="M310 350H350V330"/>
</g>
<g class="arr">
<path d="M90 37L108 47L90 57Z"/>
<path d="M175 37L185 55L165 55Z"/>
<path d="M35 95L45 113L25 113Z"/>
<path d="M355 120L337 110L337 130Z"/>
<path d="M235 155L245 173L225 173Z"/>
<path d="M65 210L85 220L65 230Z"/>
<path d="M155 210L173 200L173 220Z"/>
<path d="M200 275L190 257L210 257Z"/>
<path d="M45 325L55 343L35 343Z"/>
<path d="M95 325L105 343L85 343Z"/>
<path d="M155 355L175 365L155 375Z"/>
<path d="M285 355L295 337L305 355Z"/>
<path d="M350 350L340 332L360 332Z"/>
</g>
<path id="trail" d="M200 135V180H155"/>
<circle id="dot" cx="200" cy="135" r="5"/>
</svg>
</div>
<div id="msg">START dabao.</div>
<div>
<button id="start">▶ START</button>
<button id="restart">↻ RESTART</button>
</div>
<p>Player ko chalane ke liye arrows dabao.</p>
<div id="controls">
<span></span><button data-d="up">↑</button><span></span>
<button data-d="left">←</button><button data-d="down">↓</button><button data-d="right">→</button>
</div>
<script>
const dot=document.getElementById('dot');
const trail=document.getElementById('trail');
const msg=document.getElementById('msg');
let x=200,y=135,started=false;
const startPoint={x:200,y:135};
function draw(){
 dot.setAttribute('cx',x);dot.setAttribute('cy',y);
 trail.setAttribute('d','M'+startPoint.x+' '+startPoint.y+' L'+x+' '+y);
}
function reset(){
 x=200;y=135;started=false;draw();
 msg.textContent='START dabao.';
}
document.getElementById('start').onclick=()=>{
 started=true;msg.textContent='Ab arrows se player chalao!';
};
document.getElementById('restart').onclick=reset;
function move(d){
 if(!started){msg.textContent='Pehle START dabao!';return}
 let nx=x,ny=y,step=10;
 if(d==='up')ny-=step;
 if(d==='down')ny+=step;
 if(d==='left')nx-=step;
 if(d==='right')nx+=step;
 nx=Math.max(15,Math.min(385,nx));
 ny=Math.max(15,Math.min(385,ny));
 x=nx;y=ny;draw();
 if(x>=365&&y>=350){
 msg.textContent='Maze complete! 🎉';started=false;
 }else msg.textContent='Player position: '+x+', '+y;
}
document.querySelectorAll('[data-d]').forEach(b=>{
 b.addEventListener('click',()=>move(b.dataset.d));
});
document.getElementById('board').addEventListener('click',e=>{
 if(!started)return;
 const r=e.currentTarget.getBoundingClientRect();
 const px=(e.clientX-r.left)*400/r.width;
 const py=(e.clientY-r.top)*400/r.height;
 const dx=px-x,dy=py-y;
 if(Math.abs(dx)>Math.abs(dy))move(dx>0?'right':'left');
 else move(dy>0?'down':'up');
});
draw();
</script>
</body>
</html>
