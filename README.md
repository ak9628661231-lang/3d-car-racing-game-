
<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Arrow Escape Puzzle</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;background:#10131b;color:white;
  font-family:Arial,sans-serif;text-align:center;
  user-select:none
}
header{padding:18px 10px}
h1{margin:0 0 8px;color:#70e1ff;font-size:27px}
p{color:#c5cad5;margin:8px}
.panel{display:flex;justify-content:center;gap:12px;flex-wrap:wrap;margin:12px}
.badge{background:#222838;padding:10px 16px;border-radius:12px}
#game{
  position:relative;width:min(94vw,440px);
  aspect-ratio:1/1;margin:14px auto;
  background:#fff;border-radius:14px;overflow:hidden;
  box-shadow:0 0 25px #0008
}
canvas{width:100%;height:100%;display:block;touch-action:manipulation}
button{
  border:0;border-radius:12px;padding:13px 22px;
  margin:5px;font-size:16px;font-weight:bold;
  background:#35c9ef;color:#10131b;cursor:pointer
}
button.secondary{background:#30384a;color:white}
#message{min-height:30px;color:#ffe17b;font-weight:bold}
footer{color:#9ba4b8;padding:10px 20px 25px}
</style>
</head>
<body>

<header>
  <h1>➤ Arrow Escape</h1>
  <p>तीरों को सही क्रम में बाहर निकालो!</p>
</header>

<div class="panel">
  <div class="badge">बचे हुए तीर: <b id="left">12</b></div>
  <div class="badge">स्कोर: <b id="score">0</b></div>
  <div class="badge">लेवल: <b id="level">1</b></div>
</div>

<div id="game">
  <canvas id="board" width="600" height="600"></canvas>
</div>

<div id="message">पहले उस तीर पर टैप करो जिसका रास्ता खाली है।</div>

<div>
  <button onclick="newGame()">🔄 Restart</button>
  <button class="secondary" onclick="nextLevel()">अगला लेवल ➜</button>
</div>

<footer>
  टिप: तीर जिस दिशा में इशारा करता है, उसी तरफ बाहर निकलने का रास्ता खाली होना चाहिए।
</footer>

<script>
const canvas=document.getElementById("board");
const ctx=canvas.getContext("2d");

const N=6;
const cell=100;
let arrows=[];
let score=0;
let level=1;
let selected=-1;
let animating=null;
let won=false;

// हर तीर एक खाने में रखा है।
// दिशा: R=दायाँ, L=बायाँ, U=ऊपर, D=नीचे
const layouts=[
[
 [0,0,'R'],[1,0,'D'],[2,0,'D'],[3,0,'D'],
 [4,0,'D'],[5,0,'D'],[0,1,'D'],[2,1,'R'],
 [4,1,'L'],[1,2,'R'],[3,2,'D'],[5,2,'L'],
 [0,3,'R'],[2,3,'U'],[4,3,'R'],[1,4,'U'],
 [3,4,'L'],[5,4,'U'],[0,5,'U'],[2,5,'R'],
 [4,5,'R'],[5,5,'U']
],
[
 [0,0,'D'],[1,0,'R'],[3,0,'D'],[5,0,'D'],
 [0,1,'R'],[2,1,'D'],[4,1,'L'],[1,2,'U'],
 [3,2,'R'],[5,2,'L'],[0,3,'D'],[2,3,'R'],
 [4,3,'D'],[1,4,'R'],[3,4,'U'],[5,4,'U'],
 [0,5,'R'],[2,5,'U'],[4,5,'R']
]
];

function makeArrows(){
  const layout=layouts[(level-1)%layouts.length];
  arrows=layout.map((a,i)=>({
    x:a[0],y:a[1],dir:a[2],id:i,alive:true,
    progress:0
  }));
  selected=-1;
  animating=null;
  won=false;
  updateUI();
  draw();
  message("पहले खाली रास्ते वाला तीर चुनो।");
}

function updateUI(){
  document.getElementById("left").textContent=
    arrows.filter(a=>a.alive).length;
  document.getElementById("score").textContent=score;
  document.getElementById("level").textContent=level;
}

function message(t){
  document.getElementById("message").textContent=t;
}

function directionVector(d){
  return {
    R:[1,0],L:[-1,0],U:[0,-1],D:[0,1]
  }[d];
}

function canExit(a){
  const [dx,dy]=directionVector(a.dir);
  let x=a.x+dx,y=a.y+dy;

  while(x>=0&&x<N&&y>=0&&y<N){
    const blocker=arrows.find(b=>
      b.alive&&b.id!==a.id&&b.x===x&&b.y===y
    );
    if(blocker)return false;
    x+=dx;
    y+=dy;
  }
  return true;
}

function drawArrow(a){
  const cx=a.x*cell+cell/2;
  const cy=a.y*cell+cell/2;
  const [dx,dy]=directionVector(a.dir);
  const angle=Math.atan2(dy,dx);

  ctx.save();
  ctx.translate(cx,cy);

  if(animating&&animating.id===a.id){
    ctx.translate(
      dx*animating.progress*cell*0.95,
      dy*animating.progress*cell*0.95
    );
  }

  ctx.rotate(angle);

  ctx.lineCap="round";
  ctx.lineJoin="round";
  ctx.strokeStyle=a.id===selected?"#ef3340":"#111";
  ctx.fillStyle=ctx.strokeStyle;
  ctx.lineWidth=10;

  ctx.beginPath();
  ctx.moveTo(-30,0);
  ctx.lineTo(22,0);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(15,-17);
  ctx.lineTo(39,0);
  ctx.lineTo(15,17);
  ctx.closePath();
  ctx.fill();

  ctx.restore();
}

function draw(){
  ctx.clearRect(0,0,600,600);
  ctx.fillStyle="#fff";
  ctx.fillRect(0,0,600,600);

  // हल्का ग्रिड
  ctx.strokeStyle="#eeeeee";
  ctx.lineWidth=1;
  for(let i=1;i<N;i++){
    ctx.beginPath();
    ctx.moveTo(i*cell,0);
    ctx.lineTo(i*cell,600);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(0,i*cell);
    ctx.lineTo(600,i*cell);
    ctx.stroke();
  }

  arrows.forEach(a=>{
    if(a.alive)drawArrow(a);
  });

  if(won){
    ctx.fillStyle="#000a";
    ctx.fillRect(0,0,600,600);
    ctx.fillStyle="#fff";
    ctx.textAlign="center";
    ctx.font="bold 42px Arial";
    ctx.fillText("LEVEL COMPLETE!",300,270);
    ctx.font="24px Arial";
    ctx.fillText("🎉 शानदार!",300,325);
  }
}

function tap(e){
  if(won||animating)return;

  const rect=canvas.getBoundingClientRect();
  const x=(e.clientX-rect.left)*600/rect.width;
  const y=(e.clientY-rect.top)*600/rect.height;

  const gx=Math.floor(x/cell);
  const gy=Math.floor(y/cell);

  let a=arrows.find(b=>
    b.alive&&b.x===gx&&b.y===gy
  );

  if(!a){
    message("किसी तीर पर टैप करो।");
    return;
  }

  selected=a.id;

  if(!canExit(a)){
    message("❌ रास्ता बंद है! पहले दूसरे तीर हटाओ।");
    draw();
    return;
  }

  message("✅ सही रास्ता! तीर बाहर जा रहा है।");
  animating={id:a.id,progress:0};
  let start=null;

  function frame(time){
    if(!start)start=time;
    animating.progress=Math.min((time-start)/350,1);
    draw();

    if(animating.progress<1){
      requestAnimationFrame(frame);
    }else{
      a.alive=false;
      animating=null;
      selected=-1;
      score+=10;
      updateUI();

      if(arrows.every(b=>!b.alive)){
        won=true;
        score+=50;
        updateUI();
        message("🎉 जीत गए! अगले लेवल पर जाने के लिए बटन दबाओ।");
      }else{
        message("अच्छा! अब अगला खाली रास्ता खोजो।");
      }
      draw();
    }
  }

  requestAnimationFrame(frame);
}

canvas.addEventListener("pointerdown",tap);

function newGame(){
  score=0;
  level=1;
  makeArrows();
}

function nextLevel(){
  level++;
  makeArrows();
  message("लेवल "+level+" शुरू! सारे तीर बाहर निकालो।");
}

makeArrows();
</script>
</body>
</html>
