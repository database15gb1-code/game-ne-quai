<!doctype html>
<html lang="vi">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Né Quái - Minecraft Demo</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;
  background:#101827;
  color:#fff;
  font-family:Arial,sans-serif;
  text-align:center
}
.wrap{
  max-width:760px;
  margin:20px auto;
  padding:12px
}
.top{
  display:flex;
  justify-content:space-between;
  gap:10px;
  flex-wrap:wrap;
  margin-bottom:10px
}
canvas{
  width:100%;
  max-width:720px;
  border:2px solid #394968;
  border-radius:12px;
  background:#18243a;
  touch-action:none
}
button{
  font-size:20px;
  padding:12px 22px;
  margin:8px 4px;
  border:0;
  border-radius:10px;
  cursor:pointer
}
#start{background:#67d36b}
.move{background:#dce6f5}
.small{
  opacity:.75;
  font-size:14px
}
</style>
</head>

<body>

<div class="wrap">

<h1>🎮 Né Quái</h1>

<div class="top">
  <b>Điểm: <span id="score">0</span></b>
  <b>❤️ Máu: <span id="hp">3</span></b>
  <b>✨ Skill: <span id="skill">Không</span></b>
</div>

<canvas id="game" width="720" height="420"></canvas>

<div>
  <button id="start">▶ Bắt đầu</button>
</div>

<div>
  <button class="move" id="left">←</button>
  <button class="move" id="right">→</button>
</div>

<p class="small">
PC: dùng ← → hoặc A/D. Điện thoại: giữ nút ← →.
Ăn 🍎 để nhận skill.
</p>

</div>

<script>

const c=document.getElementById("game");
const x=c.getContext("2d");

const scoreEl=document.getElementById("score");
const hpEl=document.getElementById("hp");
const skillEl=document.getElementById("skill");

let p;
let enemies;
let bombs;
let fruits;
let bullets;
let particles;

let score;
let hp;
let running=false;
let last;

let spawn;
let bombSpawn;
let fruitSpawn;
let shootTimer;

let skill="Không";
let skillTime=0;

let keys={};
let raf;

const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));


// =========================
// RESET GAME
// =========================

function reset(){

  p={
    x:360,
    y:365,
    w:42,
    h:24,
    s:320
  };

  enemies=[];
  bombs=[];
  fruits=[];
  bullets=[];
  particles=[];

  score=0;
  hp=3;

  spawn=0;

  // Bom xuất hiện lần đầu sau 5 giây
  bombSpawn=5;

  // Trái cây xuất hiện lần đầu sau 7 giây
  fruitSpawn=7;

  shootTimer=.3;

  // Ban đầu KHÔNG có skill
  skill="Không";
  skillTime=0;

  scoreEl.textContent=0;
  hpEl.textContent=3;
  skillEl.textContent="Không";

  draw();
}


// =========================
// KIỂM TRA VA CHẠM
// =========================

function hit(a,b){

  return Math.abs(a.x-b.x)<(a.w+b.s)/2 &&
         Math.abs(a.y-b.y)<(a.h+b.s)/2;

}


// =========================
// HIỆU ỨNG NỔ
// =========================

function boom(px,py){

  for(let i=0;i<20;i++){

    particles.push({
      x:px,
      y:py,
      vx:(Math.random()-.5)*180,
      vy:(Math.random()-.5)*180,
      life:.6
    });

  }

}


// =========================
// VẼ GAME
// =========================

function draw(){

  x.clearRect(0,0,720,420);

  // nền
  x.fillStyle="#18243a";
  x.fillRect(0,0,720,420);

  // đường nền
  x.fillStyle="#22324d";

  for(let i=0;i<14;i++){
    x.fillRect(i*60,0,1,420);
  }

  // mặt đất
  x.fillStyle="#4d7c59";
  x.fillRect(0,395,720,25);

  // người chơi
  x.fillStyle="#77b255";
  x.fillRect(
    p.x-p.w/2,
    p.y-p.h/2,
    p.w,
    p.h
  );

  // mắt
  x.fillStyle="#fff";
  x.fillRect(p.x-12,p.y-5,7,7);
  x.fillRect(p.x+5,p.y-5,7,7);


  // =========================
  // ĐẠN
  // =========================

  bullets.forEach(b=>{

    x.fillStyle="#ffe66b";

    x.fillRect(
      b.x-3,
      b.y-8,
      6,
      12
    );

  });


  // =========================
  // QUÁI
  // =========================

  enemies.forEach(e=>{

    x.fillStyle="#d94f4f";

    x.fillRect(
      e.x-e.s/2,
      e.y-e.s/2,
      e.s,
      e.s
    );

    x.fillStyle="#111";

    x.fillRect(
      e.x-9,
      e.y-7,
      6,
      6
    );

    x.fillRect(
      e.x+3,
      e.y-7,
      6,
      6
    );

  });


  // =========================
  // BOM
  // =========================

  bombs.forEach(b=>{

    x.fillStyle="#252525";

    x.beginPath();

    x.arc(
      b.x,
      b.y,
      b.r,
      0,
      Math.PI*2
    );

    x.fill();

    // dây bom
    x.fillStyle="#ffb52e";

    x.fillRect(
      b.x-2,
      b.y-b.r-7,
      4,
      7
    );

  });


  // =========================
  // TRÁI CÂY
  // =========================

  fruits.forEach(f=>{

    x.font="27px Arial";
    x.textAlign="center";

    x.fillText(
      "🍎",
      f.x,
      f.y+9
    );

  });


  // =========================
  // KHIÊN
  // =========================

  if(skill==="Khiên" && skillTime>0){

    x.strokeStyle="#72edff";
    x.lineWidth=4;

    x.beginPath();

    x.arc(
      p.x,
      p.y,
      32,
      0,
      Math.PI*2
    );

    x.stroke();

  }


  // =========================
  // HIỆU ỨNG
  // =========================

  particles.forEach(q=>{

    x.fillStyle="#ffd166";

    x.fillRect(
      q.x,
      q.y,
      4,
      4
    );

  });


  x.fillStyle="#fff";
  x.font="16px Arial";
  x.textAlign="left";

  x.fillText(
    "Né quái càng lâu càng nhiều điểm!",
    18,
    28
  );

}


// =========================
// GAME LOOP
// =========================

function loop(now){

  if(!running)return;

  let dt=Math.min(
    .033,
    (now-last)/1000
  );

  last=now;


  // =========================
  // DI CHUYỂN
  // =========================

  if(keys.left || keys.a)
    p.x-=p.s*dt;

  if(keys.right || keys.d)
    p.x+=p.s*dt;

  p.x=clamp(
    p.x,
    25,
    695
  );


  // =========================
  // THỜI GIAN
  // =========================

  score+=dt;

  spawn-=dt;
  bombSpawn-=dt;
  fruitSpawn-=dt;
  shootTimer-=dt;

  // skill giảm thời gian
  if(skill!=="Không"){

    skillTime-=dt;

    if(skillTime<=0){

      skill="Không";
      skillTime=0;

      skillEl.textContent="Không";

    }

  }


  // =========================
  // SINH QUÁI
  // =========================

  if(spawn<=0){

    enemies.push({

      x:30+Math.random()*660,

      y:-25,

      s:28+Math.random()*18,

      v:
        100+
        Math.random()*100+
        score*1.5

    });

    spawn=Math.max(
      .25,
      .75-score*.003
    );

  }


  // =========================
  // BOM
  // =========================

  if(bombSpawn<=0){

    bombs.push({

      x:30+Math.random()*660,

      y:-30,

      r:15,

      v:180+Math.random()*80

    });

    bombSpawn=
      7+
      Math.random()*5;

  }


  // =========================
  // TRÁI CÂY
  // =========================

  if(fruitSpawn<=0){

    fruits.push({

      x:35+Math.random()*650,

      y:-25,

      r:15,

      v:65

    });

    fruitSpawn=
      8+
      Math.random()*7;

  }


  // =========================
  // BẮN
  // =========================

  // Bình thường bắn chậm
  // Bắn nhanh thì bắn rất nhanh

  let rate=
    skill==="Bắn nhanh"
    ? .12
    : .45;


  if(shootTimer<=0){

    bullets.push({

      x:p.x,

      y:p.y-22,

      r:5,

      v:430

    });

    shootTimer=rate;

  }


  // =========================
  // QUÁI DI CHUYỂN
  // =========================

  for(let i=enemies.length-1;i>=0;i--){

    let e=enemies[i];

    e.y+=e.v*dt;


    if(hit(p,e)){

      // Khiên chắn quái
      if(
        skill==="Khiên" &&
        skillTime>0
      ){

        boom(
          e.x,
          e.y
        );

        enemies.splice(i,1);

        score+=5;

        continue;

      }


      // Không có khiên
      enemies.splice(i,1);

      hp--;

      hpEl.textContent=hp;


      if(hp<=0){

        running=false;

        alert(
          "Game Over! Điểm: "+
          Math.floor(score)
        );

        return;

      }

    }

    else if(e.y>445){

      enemies.splice(i,1);

    }

  }


  // =========================
  // BOM
  // =========================

  for(let i=bombs.length-1;i>=0;i--){

    let b=bombs[i];

    b.y+=b.v*dt;

    let exploded=false;


    // Bom đụng quái
    for(
      let j=enemies.length-1;
      j>=0;
      j--
    ){

      if(
        Math.hypot(
          b.x-enemies[j].x,
          b.y-enemies[j].y
        )
        <
        b.r+
        enemies[j].s/2+
        12
      ){

        boom(
          b.x,
          b.y
        );

        enemies.splice(j,1);

        score+=10;

        exploded=true;

      }

    }


    // Bom đụng người chơi
    if(
      Math.hypot(
        b.x-p.x,
        b.y-p.y
      )
      <
      b.r+20
    ){

      // Khiên chắn bom
      if(
        skill==="Khiên" &&
        skillTime>0
      ){

        boom(
          b.x,
          b.y
        );

        bombs.splice(i,1);

        continue;

      }


      hp--;

      hpEl.textContent=hp;

      boom(
        b.x,
        b.y
      );

      bombs.splice(i,1);


      if(hp<=0){

        running=false;

        alert(
          "Game Over! Điểm: "+
          Math.floor(score)
        );

        return;

      }

    }

    else if(
      exploded ||
      b.y>450
    ){

      bombs.splice(i,1);

    }

  }


  // =========================
  // ĂN TRÁI CÂY
  // =========================

  for(let i=fruits.length-1;i>=0;i--){

    let f=fruits[i];

    f.y+=f.v*dt;


    if(
      Math.hypot(
        f.x-p.x,
        f.y-p.y
      )
      <
      f.r+22
    ){

      // Chọn ngẫu nhiên 1 skill
      const list=[
        "Bắn nhanh",
        "Một tia",
        "Khiên"
      ];

      skill=
        list[
          Math.floor(
            Math.random()*list.length
          )
        ];


      // Skill tồn tại đúng 20 giây
      skillTime=20;


      skillEl.textContent=
        skill+" 20s";


      fruits.splice(i,1);

    }

    else if(f.y>450){

      fruits.splice(i,1);

    }

  }


  // =========================
  // ĐẠN
  // =========================

  for(let i=bullets.length-1;i>=0;i--){

    let b=bullets[i];

    b.y-=b.v*dt;


    if(b.y<-20){

      bullets.splice(i,1);

      continue;

    }


    for(
      let j=enemies.length-1;
      j>=0;
      j--
    ){

      if(
        Math.hypot(
          b.x-enemies[j].x,
          b.y-enemies[j].y
        )
        <
        b.r+
        enemies[j].s/2
      ){

        boom(
          enemies[j].x,
          enemies[j].y
        );

        enemies.splice(j,1);

        bullets.splice(i,1);

        score+=5;

        break;

      }

    }

  }


  // =========================
  // PARTICLES
  // =========================

  particles.forEach(q=>{

    q.x+=q.vx*dt;

    q.y+=q.vy*dt;

    q.life-=dt;

  });


  particles=
    particles.filter(
      q=>q.life>0
    );


  // =========================
  // HIỂN THỊ
  // =========================

  scoreEl.textContent=
    Math.floor(score);


  if(skill!=="Không"){

    skillEl.textContent=
      skill+
      " "+
      Math.ceil(skillTime)+
      "s";

  }


  draw();

  raf=
    requestAnimationFrame(loop);

}


// =========================
// BẮT ĐẦU
// =========================

function start(){

  cancelAnimationFrame(raf);

  reset();

  running=true;

  last=
    performance.now();

  raf=
    requestAnimationFrame(loop);

}

document
  .getElementById("start")
  .onclick=start;


// =========================
// NÚT DI CHUYỂN
// =========================

function hold(id,k){

  let b=
    document.getElementById(id);

  b.onpointerdown=
    ()=>{
      keys[k]=true;
    };

  b.onpointerup=
    ()=>{
      keys[k]=false;
    };

  b.onpointerleave=
    ()=>{
      keys[k]=false;
    };

  b.onpointercancel=
    ()=>{
      keys[k]=false;
    };

}

hold("left","left");
hold("right","right");


// =========================
// BÀN PHÍM
// =========================

onkeydown=e=>{

  if(e.key==="ArrowLeft")
    keys.left=true;

  if(e.key==="ArrowRight")
    keys.right=true;

  if(e.key.toLowerCase()==="a")
    keys.a=true;

  if(e.key.toLowerCase()==="d")
    keys.d=true;

};

onkeyup=e=>{

  if(e.key==="ArrowLeft")
    keys.left=false;

  if(e.key==="ArrowRight")
    keys.right=false;

  if(e.key.toLowerCase()==="a")
    keys.a=false;

  if(e.key.toLowerCase()==="d")
    keys.d=false;

};


reset();

</script>

</body>
</html>
