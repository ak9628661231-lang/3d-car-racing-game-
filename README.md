<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Arrow Maze Game</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#111;color:#fff;font-family:Arial,sans-serif;text-align:center}
h2{margin:16px 0 8px;font-size:24px}
p{margin:8px;font-size:14px;color:#ccc}
#board{width:100%;max-width:650px;margin:12px auto;background:#fff;touch-action:none}
svg{width:100%;height:auto;display:block}
.wall{fill:none;stroke:#050505;stroke-width:9;stroke-linecap:round;stroke-linejoin:round}
.arrow{fill:#050505}
#playerPath{fill:none;stroke:#f01928;stroke-width:8;stroke-linecap:round;stroke-linejoin:round}
#player{fill:#f01928;stroke:white;stroke-width:2}
button{border:0;border-radius:10px;padding:13px 22px;margin:8px;background:#f01928;color:white;font-weight:bold;font-size:16px}
button:active{transform:scale(.96)}
#status{min-height:24px;color:#fff;font-weight:bold}
</style>
</head>
<body>
<h2>ARROW MAZE</h2>
<p>Laal raaste ko follow karo aur maze poora karo!</p>
<div id="board">
<svg id="maze" viewBox="0 0 600 600" aria-label="Arrow maze game">
<defs>
<marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto">
<path d="M0,0 L8,4 L0,8 Z" class="arrow"/>
</marker>
</defs>
<rect width="600" height="600" fill="white"/>
<g class="wall">
<path d="M55 45 H195 V140 H130 V185 H195 V230 H265 V45 H325 V190"/>
<path d="M55 45 V210" marker-end="url(#arrow)"/>
<path d="M100 120 V85 H175" marker-end="url(#arrow)"/>
<path d="M100 120 V230 H165 V270 H270"/>
<path d="M295 45 V230 H330 V290 H355 V165 H385"/>
<path d="M355 45 H550 V250 H445 V315 H550"/>
<path d="M355 120 H420 V155 H455 V85 H515 V140"/>
<path d="M385 125 H425 V290" marker-end="url(#arrow)"/>
<path d="M475 110 V185 H425 V225 H390 V270"/>
<path d="M515 165 V215 H465 V285 H550"/>
<path d="M55 260 H135"/>
<path d="M55 260 V360 H175" marker-end="url(#arrow)"/>
<path d="M95 295 H220 V355"/>
<path d="M95 295 V325 H190 V355"/>
<path d="M220 355 H350 V385" marker-end="url(#arrow)"/>
<path d="M465 285 V340 H520 V430" marker-end="url(#arrow)"/>
<path d="M550 315 H575 V455 H520 V520 H325 V440 H280 V400 H245 V455 H210 V405 H180 V470 H155 V545 H55 V390 H120 V425 H95 V455"/>
<path d="M55 390 H120 V425 H95 V455"/>
<path d="M155 545 V475 H125" />
<path d="M180 470 V405 H325 V435 H355 V490"/>
<path d="M210 545 H300" marker-end="url(#arrow)"/>
<path d="M245 455 V490 H290 V505 H210 V545"/>
<path d="M405 355 H455"/>
<path d="M425 355 V520" marker-end="url(#arrow)"/>
<path d="M455 390 V455 H495"/>
<path d="M455 390 V430 H480 V480 H510"/>
<path d="M405 520 H375 V440 H405"/>
<path d="M465 520 H530 V490" marker-end="url(#arrow)"/>
<path d="M550 390 H575 V430" marker-end="url(#arrow)"/>
<path d="M100 300 H220"/>
</g>
<!-- Arrows in the original maze -->
<g class="arrow">
<path d="M155 72 L185 88 L155 104 Z"/>
<path d="M280 65 L296 95 L264 95 Z"/>
<path d="M450 245 L480 229 L480 261 Z"/>
<path d="M465 320 L481 350 L449 350 Z"/>
<path d="M90 470 L106 500 L74 500 Z"/>
<path d="M150 470 L166 500 L134 500 Z"/>
<path d="M300 440 L270 456 L300 472 Z"/>
<path d="M505 500 L521 470 L489 470 Z"/>
</g>
<!-- Highlighted route -->
<path id="playerPath" d="M345 185 V295 H245" />
<path d="M245 295 L270 280 L270 310 Z" fill="#f01928"/>
<circle id="player" cx="345" cy="185" r="7"/>
</svg>
</div>
<div id="status">Start dabao aur laal raasta dekho.</div>
<button id="start">▶ START</button>
<button id="reset">↻ RESTART</button>
<script>
const path=document.getElementById('playerPath');
const player=document.getElementById('player');
const statusText=document.getElementById('status');
const start=document.getElementById('start');
let animation=null;
function reset(){
 if(animation) cancelAnimationFrame(animation);
 animation=null;
 player.setAttribute('cx',345);
 player.setAttribute('cy',185);
 path.style.strokeDasharray='none';
 statusText.textContent='Start dabao aur laal raasta dekho.';
 start.disabled=false;
}
start.addEventListener('click',()=>{
 if(animation) cancelAnimationFrame(animation);
 start.disabled=true;
 statusText.textContent='Laal raasta chal raha hai...';
 const length=path.getTotalLength();
 const duration=2500;
 const begin=performance.now();
 path.style.strokeDasharray=length;
 path.style.strokeDashoffset=length;
 function step(now){
  const t=Math.min((now-begin)/duration,1);
  const point=path.getPointAtLength(t*length);
  player.setAttribute('cx',point.x);
  player.setAttribute('cy',point.y);
  path.style.strokeDashoffset=length*(1-t);
  if(t<1){animation=requestAnimationFrame(step)}
  else{
   animation=null;
   statusText.textContent='Raasta poora ho gaya! 🎉';
   start.disabled=false;
  }
 }
 animation=requestAnimationFrame(step);
});
document.getElementById('reset').addEventListener('click',reset);
</script>
</body>
</html>
