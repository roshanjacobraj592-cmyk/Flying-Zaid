<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Flying Zaid</title>
  <style>
    html,body{height:100%;margin:0;font-family:Arial;background:#0b1220;color:#e6eef3;}
    canvas{display:block;margin:0 auto;background:#7ec0ee;}
  </style>
</head>
<body>
<canvas id="gameCanvas" width="900" height="600"></canvas>
<script>
// === Flying Zaid (Flappy-like game) ===
const canvas=document.getElementById('gameCanvas');
const ctx=canvas.getContext('2d');
let W=canvas.width,H=canvas.height;

// Game state
let bird={x:150,y:H/2,vy:0,width:44,height:32};
let pillars=[],lastSpawn=0,score=0,running=false,gameOver=false,lastTs=0;

// Settings
let settings={gravity:800,jumpPower:360,pillarGap:160,pillarSpeed:210,spawnInterval:1500};

// Placeholder assets
function createPlaceholder(w,h,text){const c=document.createElement('canvas');c.width=w;c.height=h;const g=c.getContext('2d');g.fillStyle='#7fb3e6';g.fillRect(0,0,w,h);g.fillStyle='#fff';g.font='bold 18px sans-serif';g.textAlign='center';g.fillText(text,w/2,h/2);const img=new Image();img.src=c.toDataURL();return img;}
let charImg=createPlaceholder(44,32,'ZAID');
let bgImg=createPlaceholder(900,600,'BACKGROUND');
let pillarImg=createPlaceholder(80,400,'PILLAR');

// Jump & death sounds
const audioCtx=new(window.AudioContext||window.webkitAudioContext)();
let jumpSound=()=>{const o=audioCtx.createOscillator();const g=audioCtx.createGain();o.type='sine';o.frequency.value=880;g.gain.value=0.05;o.connect(g);g.connect(audioCtx.destination);o.start();g.gain.exponentialRampToValueAtTime(0.0001,audioCtx.currentTime+0.08);o.stop(audioCtx.currentTime+0.1);};
let deathSound=()=>{const o=audioCtx.createOscillator();const g=audioCtx.createGain();o.type='sine';o.frequency.value=120;g.gain.value=0.05;o.connect(g);g.connect(audioCtx.destination);o.start();g.gain.exponentialRampToValueAtTime(0.0001,audioCtx.currentTime+0.35);o.stop(audioCtx.currentTime+0.37);};

// Game mechanics
function spawnPillar(){const x=W+60;const gap=settings.pillarGap;const topH=80+Math.random()*(H-gap-160);pillars.push({x:x,topH:topH,bottomY:topH+gap,width:Math.max(60,pillarImg.width*0.5)});}
function resetGame(){bird={x:150,y:H/2,vy:0,width:charImg.width,height:charImg.height};pillars=[];lastSpawn=0;score=0;gameOver=false;}
function startGame(){running=true;resetGame();lastTs=performance.now();requestAnimationFrame(loop);}
function endGame(){gameOver=true;running=false;deathSound();}
function update(dt){if(gameOver)return;bird.vy+=settings.gravity*dt;bird.y+=bird.vy*dt;if(bird.y+bird.height/2>=H){bird.y=H-bird.height/2;endGame();}if(bird.y-bird.height/2<=0){bird.y=bird.height/2;bird.vy=0;}lastSpawn+=dt*1000;if(lastSpawn>settings.spawnInterval){spawnPillar();lastSpawn=0;}for(let i=pillars.length-1;i>=0;i--){const p=pillars[i];p.x-=settings.pillarSpeed*dt;if(!p.passed&&p.x+p.width<bird.x){score++;p.passed=true;}if(bird.x+bird.width/2>p.x&&bird.x-bird.width/2<p.x+p.width){if(bird.y-bird.height/2<p.topH||bird.y+bird.height/2>p.bottomY){endGame();}}if(p.x+p.width<-200)pillars.splice(i,1);}}
function drawBackground(){const scale=Math.max(W/bgImg.width,H/bgImg.height);const imgW=bgImg.width*scale,imgH=bgImg.height*scale;const t=performance.now()/6000;const offset=(t*30)%imgW;for(let x=-imgW;x<W+imgW;x+=imgW){ctx.drawImage(bgImg,x-offset,0,imgW,imgH);}}
function drawPillars(){for(const p of pillars){const w=p.width,topH=p.topH,bottomY=p.bottomY;const scale=w/pillarImg.width,hScaled=pillarImg.height*scale;ctx.save();ctx.translate(p.x+w/2,topH-(hScaled/2));ctx.scale(1,-1);ctx.drawImage(pillarImg,-w/2,-hScaled/2,w,hScaled);ctx.restore();ctx.drawImage(pillarImg,p.x,bottomY,w,hScaled);}}
function drawBird(){ctx.save();ctx.translate(bird.x,bird.y);ctx.rotate(Math.max(-0.6,Math.min(1.2,bird.vy/500)));ctx.drawImage(charImg,-bird.width/2,-bird.height/2,bird.width,bird.height);ctx.restore();}
function loop(ts){const dt=Math.min(0.03,(ts-lastTs)/1000);lastTs=ts;update(dt);ctx.clearRect(0,0,W,H);drawBackground();drawPillars();drawBird();ctx.fillStyle='#fff';ctx.font='20px sans-serif';ctx.fillText('Score: '+score,18,34);if(!running){ctx.fillStyle='rgba(0,0,0,0.5)';ctx.fillRect(W/2-160,H/2-54,320,108);ctx.fillStyle='#fff';ctx.textAlign='center';ctx.fillText('Click / Space to start',W/2,H/2-12);ctx.textAlign='left';}if(gameOver){ctx.fillStyle='rgba(0,0,0,0.6)';ctx.fillRect(W/2-200,H/2-90,400,180);ctx.fillStyle='#fff';ctx.font='28px sans-serif';ctx.textAlign='center';ctx.fillText('Game Over',W/2,H/2-20);ctx.font='18px sans-serif';ctx.fillText('Score: '+score,W/2,H/2+12);ctx.fillText('Press any key to restart',W/2,H/2+50);ctx.textAlign='left';}if(running)requestAnimationFrame(loop);}
function jump(){if(!running){startGame();return;}if(gameOver)return;bird.vy=-settings.jumpPower;jumpSound();}
window.addEventListener('keydown',e=>{if(e.code==='Space'){e.preventDefault();jump();}});
canvas.addEventListener('mousedown',()=>jump());canvas.addEventListener('touchstart',e=>{e.preventDefault();jump();},{passive:false});
startGame();
</script>
</body>
</html>
