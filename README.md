# Chesslap
<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#10151d">
<title>Chess Lab — AI Satranç</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>

<style>
:root{
--bg:#10151d;
--panel:#1a2330;
--panel2:#202c3b;
--text:#eef3f8;
--muted:#9eacbd;
--accent:#71d6b1;
--light:#e8edf2;
--dark:#668078;
--gold:#f4c96b
}

*{box-sizing:border-box}

body{
margin:0;
background:radial-gradient(circle at 15% 0%,#26384a 0,#10151d 42%);
color:var(--text);
font:15px system-ui,Segoe UI,sans-serif;
min-height:100vh
}

header{
padding:22px max(20px,calc((100vw - 1100px)/2));
display:flex;
justify-content:space-between;
align-items:center;
border-bottom:1px solid #ffffff14
}

.brand{font-size:22px;font-weight:800;letter-spacing:.3px}
.brand span{color:var(--accent)}
.tag{color:var(--muted);font-size:13px}

main{
max-width:1100px;
margin:28px auto;
padding:0 18px;
display:grid;
grid-template-columns:minmax(300px,620px) minmax(270px,1fr);
gap:22px
}

.card{
background:#1a2330eF;
border:1px solid #ffffff14;
border-radius:18px;
padding:18px;
box-shadow:0 18px 55px #0003
}

.boardtop{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:12px
}

.turn{color:var(--muted);font-size:13px}

#board{
display:grid;
grid-template-columns:repeat(8,1fr);
aspect-ratio:1;
border:5px solid #344353;
border-radius:8px;
overflow:hidden
}

.sq{
display:flex;
align-items:center;
justify-content:center;
position:relative;
font-size:clamp(26px,5vw,56px);
cursor:pointer;
user-select:none;
line-height:1
}

.sq.light{background:var(--light)}
.sq.dark{background:var(--dark)}
.sq.selected{box-shadow:inset 0 0 0 5px var(--gold)}
.sq.last{background:#d9c36b!important}

.sq.hint:after{
content:"";
width:23%;
height:23%;
border-radius:50%;
background:#16222b66;
position:absolute
}

.sq.capture:after{
content:"";
position:absolute;
inset:4px;
border:5px solid #16222b66;
border-radius:50%
}

.sq .coord{
position:absolute;
bottom:3px;
left:4px;
font:10px system-ui;
color:#18232d88
}

.pieces{filter:drop-shadow(0 2px 1px #0003)}

.controls{
display:flex;
gap:8px;
flex-wrap:wrap;
margin-top:14px
}

button,select{
border:1px solid #ffffff20;
border-radius:10px;
background:var(--panel2);
color:var(--text);
padding:10px 13px;
font:inherit;
cursor:pointer
}

button.primary{
background:var(--accent);
color:#102019;
border:0;
font-weight:750
}

button:hover{filter:brightness(1.12)}
button:disabled{opacity:.45;cursor:not-allowed}

.side{display:flex;flex-direction:column;gap:14px}
.section-title{font-size:16px;font-weight:750;margin:0 0 12px}

.row{
display:flex;
align-items:center;
justify-content:space-between;
gap:10px;
margin:10px 0
}

.row label{color:var(--muted)}
select{max-width:160px}

.suggestion{
background:#132c2a;
border:1px solid #3a806e;
border-radius:12px;
padding:14px;
margin-top:12px;
min-height:78px
}

.suggestion strong{color:var(--accent)}
.small{font-size:12px;color:var(--muted);line-height:1.5}

#moves{
height:155px;
overflow:auto;
background:#111923;
border-radius:10px;
padding:10px;
font-family:ui-monospace,monospace;
color:#cbd6e2;
line-height:1.8
}

.status{
padding:10px 12px;
border-radius:10px;
background:#ffffff0a;
color:var(--muted);
font-size:13px;
margin-top:12px
}

footer{
text-align:center;
color:var(--muted);
font-size:12px;
padding:18px
}

@media(max-width:800px){
main{
grid-template-columns:1fr;
margin:10px auto;
padding:0 10px;
gap:12px
}

.card{padding:12px;border-radius:14px}
header{padding:14px 14px}
.tag{display:none}

#board{
width:100%;
max-width:min(94vw,620px);
margin:0 auto;
border-width:3px
}

.sq{
font-size:clamp(29px,10vw,58px);
touch-action:manipulation
}

.controls{
display:grid;
grid-template-columns:repeat(2,minmax(0,1fr));
gap:8px
}

.controls button{
min-height:46px;
padding:10px 7px;
font-size:14px
}

.boardtop{font-size:14px}
.side{gap:10px}
select{max-width:58%;min-height:42px}
button{min-height:42px;touch-action:manipulation}
#moves{height:125px}
}

@media(max-width:380px){
.sq{font-size:clamp(25px,9vw,36px)}
.brand{font-size:19px}
}
</style>
</head>
<body>

<header>
<div class="brand">♟ Chess <span>Lab</span></div>
<div class="tag">İki tarafı da sen yönet · Yapay zekâdan taktik önerileri al</div>
</header>

<main>
<section class="card">
<div class="boardtop">
<strong>Satranç tahtası</strong>
<span class="turn" id="turn">Sıra: Beyaz</span>
</div>

<div class="status" style="margin:0 0 12px">
🎮 <strong>İki taraf da senin kontrolünde:</strong>
Beyaz ve siyah taşları sen hareket ettirirsin.
Yapay zekâ yalnızca hamle önerir.
</div>

<div id="board" aria-label="Satranç tahtası"></div>

<div class="controls">
<button class="primary" id="hint">✦ En iyi hamleyi öner</button>
<button id="undo">↶ Geri al</button>
<button id="flip">Tahtayı çevir</button>
<button id="reset">Yeni oyun</button>
</div>

<div class="status" id="status">
Bir taş seç, sonra gitmesini istediğin kareye dokun.
Beyazı da siyahı da sen oynatabilirsin.
</div>
</section>

<aside class="side">
<section class="card">
<h2 class="section-title">AI hamle danışmanı</h2>

<div class="row">
<label for="level">Zorluk</label>
<select id="level">
<option value="1">Kolay</option>
<option value="2">Orta</option>
<option value="3" selected>Biraz zor</option>
<option value="4">Zor</option>
<option value="5">Usta</option>
</select>
</div>

<div class="row">
<label for="human">Oyun tarzı</label>
<select id="human">
<option value="yes" selected>İnsan gibi</option>
<option value="no">En güçlü hamle</option>
</select>
</div>

<div class="suggestion" id="advice">
<strong>Hazır!</strong>
<div class="small">
Tahtadaki konumu analiz etmek için
“En iyi hamleyi öner” düğmesine bas.
</div>
</div>

<p class="small">
Öneriler mevcut konuma göre hesaplanır.
İnsan gibi modda AI bazen en iyi yerine
güçlü ama ikinci veya üçüncü sıradaki
bir hamleyi seçebilir.
</p>
</section>

<section class="card">
<h2 class="section-title">Hamle geçmişi</h2>
<div id="moves">Henüz hamle yok.</div>
</section>

<section class="card">
<h2 class="section-title">Taktik ipuçları</h2>
<div class="small">
• Taşların korunup korunmadığını kontrol et.<br>
• Şah çekişleri ve taş alma tehditlerine bak.<br>
• Rakibin bir sonraki hamlede ne tehdit ettiğini düşün.<br>
• Öneriyi uygulamak zorunda değilsin.
</div>
</section>
</aside>
</main>

<footer>
Chess Lab · Analizler eğitim ve pratik amaçlıdır.
</footer> <script>

// Satranç motoru yüklenemezse temel motoru kullan.
if(typeof Chess==='undefined'){
 class LocalChess{
  constructor(){this.reset()}

  reset(){
   this.b=Array.from({length:8},()=>Array(8).fill(null));
   const back='rnbqkbnr';

   for(let f=0;f<8;f++){
    this.b[0][f]={type:back[f],color:'b'};
    this.b[1][f]={type:'p',color:'b'};
    this.b[6][f]={type:'p',color:'w'};
    this.b[7][f]={type:back[f],color:'w'};
   }

   this.t='w';
   this.h=[];
   this.last=null;
  }

  board(){
   return this.b.map(r=>r.map(p=>p?{...p}:null));
  }

  get(s){
   const q=this.sq(s);
   return this.b[q.r]?.[q.f]?{...this.b[q.r][q.f]}:null;
  }

  turn(){return this.t}

  sq(s){
   return {r:8-Number(s[1]),f:s.charCodeAt(0)-97};
  }

  name(r,f){
   return 'abcdefgh'[f]+(8-r);
  }

  moves(opts={}){
   let out=[];

   for(let r=0;r<8;r++){
    for(let f=0;f<8;f++){
     let p=this.b[r][f];

     if(!p||p.color!==this.t)continue;
     if(opts.square&&this.name(r,f)!==opts.square)continue;

     const add=(rr,ff)=>{
      if(rr<0||rr>7||ff<0||ff>7)return false;

      let q=this.b[rr][ff];
      if(q&&q.color===p.color)return false;

      out.push({
       from:this.name(r,f),
       to:this.name(rr,ff),
       piece:p.type,
       color:p.color,
       captured:q?.type||null,
       promotion:p.type==='p'&&(rr===0||rr===7)?'q':undefined,
       san:(q?'x':'')+this.name(rr,ff)
      });

      return !q;
     };

     if(p.type==='p'){
      let d=p.color==='w'?-1:1;
      let start=p.color==='w'?6:1;

      if(!this.b[r+d]?.[f]){
       add(r+d,f);

       if(r===start&&!this.b[r+2*d][f]){
        add(r+2*d,f);
       }
      }

      for(let df of [-1,1]){
       let q=this.b[r+d]?.[f+df];

       if(q&&q.color!==p.color){
        add(r+d,f+df);
       }
      }
     }

     else if(p.type==='n'){
      for(let [dr,df] of [
       [-2,-1],[-2,1],[-1,-2],[-1,2],
       [1,-2],[1,2],[2,-1],[2,1]
      ]){
       add(r+dr,f+df);
      }
     }

     else{
      let dirs=p.type==='b'
       ?[[1,1],[1,-1],[-1,1],[-1,-1]]
       :p.type==='r'
       ?[[1,0],[-1,0],[0,1],[0,-1]]
       :[[1,1],[1,-1],[-1,1],[-1,-1],
         [1,0],[-1,0],[0,1],[0,-1]];

      for(let [dr,df] of dirs){
       for(let n=1;n<8;n++){
        if(!add(r+dr*n,f+df*n))break;
        if(p.type==='k')break;
       }
      }
     }
    }
   }

   return out;
  }

  move(m){
   let moves=this.moves();

   let x=m.from&&m.to
    ?moves.find(z=>z.from===m.from&&z.to===m.to)
    :moves.find(z=>z.san===m);

   if(!x)return null;

   let a=this.sq(x.from);
   let b=this.sq(x.to);
   let piece=this.b[a.r][a.f];

   this.h.push({
    b:this.board(),
    t:this.t,
    last:this.last
   });

   this.b[b.r][b.f]=
    piece.type==='p'&&(b.r===0||b.r===7)
     ?{type:'q',color:piece.color}
     :piece;

   this.b[a.r][a.f]=null;
   this.t=this.t==='w'?'b':'w';
   this.last=x;

   return {...x,from:x.from,to:x.to};
  }

  undo(){
   let x=this.h.pop();

   if(x){
    this.b=x.b;
    this.t=x.t;
    this.last=x.last;
   }

   return x||null;
  }

  history(opts){
   return opts?.verbose
    ?this.h.map(x=>x.last).filter(Boolean)
    :this.h.map(x=>x.last?.san||'');
  }

  game_over(){return this.moves().length===0}
  in_checkmate(){return false}
  in_draw(){return this.game_over()}
 }

 window.Chess=LocalChess;
}

const game=new Chess();

const glyph={
 wk:"♔",wq:"♕",wr:"♖",wb:"♗",wn:"♘",wp:"♙",
 bk:"♚",bq:"♛",br:"♜",bb:"♝",bn:"♞",bp:"♟"
};

let selected=null;
let legal=[];
let flipped=false;
let lastMove=null;

const board=document.getElementById('board');
const statusEl=document.getElementById('status');

function render(){
 board.innerHTML='';

 const files=flipped
  ?['h','g','f','e','d','c','b','a']
  :['a','b','c','d','e','f','g','h'];

 const ranks=flipped
  ?[1,2,3,4,5,6,7,8]
  :[8,7,6,5,4,3,2,1];

 ranks.forEach((r,ri)=>{
  files.forEach((f,fi)=>{
   const sq=f+r;
   const el=document.createElement('div');

   el.className='sq '+((ri+fi)%2?'dark':'light');

   const p=game.get(sq);

   if(p){
    const key=p.color+p.type;
    el.innerHTML='<span class="pieces">'+glyph[key]+'</span>';
   }

   if(selected===sq)el.classList.add('selected');

   if(legal.some(m=>m.to===sq)){
    el.classList.add(game.get(sq)?'capture':'hint');
   }

   if(lastMove&&(lastMove.from===sq||lastMove.to===sq)){
    el.classList.add('last');
   }

   if(fi===0){
    el.insertAdjacentHTML(
     'beforeend',
     '<span class="coord">'+r+'</span>'
    );
   }

   el.addEventListener('click',()=>tap(sq));
   board.appendChild(el);
  });
 });

 document.getElementById('turn').textContent=
  'Sıra: '+(game.turn()==='w'?'Beyaz':'Siyah');

 document.getElementById('moves').textContent=
  game.history().length
   ?game.history({verbose:true}).reduce(
    (s,m,i)=>s+
     (i%2===0?Math.floor(i/2+1)+'. ':'')+
     m.san+
     (i%2===1?'\n':' '),
    ''
   )
   :'Henüz hamle yok.';

 if(game.game_over()){
  statusEl.textContent='Oyun sona erdi.';
 }
}

function tap(sq){
 if(game.game_over())return;

 if(selected){
  const move=game.move({
   from:selected,
   to:sq,
   promotion:'q'
  });

  if(move){
   lastMove=move;
   selected=null;
   legal=[];
   render();
   return;
  }
 }

 const p=game.get(sq);

 if(p){
  selected=sq;
  legal=game.moves({
   square:sq,
   verbose:true
  });
 }else{
  selected=null;
  legal=[];
 }

 render();
}

document.getElementById('undo').onclick=()=>{
 game.undo();
 lastMove=null;
 selected=null;
 legal=[];
 render();
 statusEl.textContent='Son hamle geri alındı.';
};

document.getElementById('reset').onclick=()=>{
 game.reset();
 lastMove=null;
 selected=null;
 legal=[];
 render();
 statusEl.textContent='Yeni oyun başladı.';
};

document.getElementById('flip').onclick=()=>{
 flipped=!flipped;
 render();
};

function val(p){
 return ({
  p:100,
  n:320,
  b:335,
  r:500,
  q:900,
  k:20000
 })[p.type];
}

function evaluate(){
 let s=0;

 for(let r=0;r<8;r++){
  for(let f=0;f<8;f++){
   let p=game.board()[r][f];

   if(p){
    s+=(p.color==='w'?1:-1)*val(p);
   }
  }
 }

 return s*(game.turn()==='w'?1:-1);
}

function search(depth,alpha,beta){
 if(depth===0||game.game_over()){
  return evaluate();
 }

 let moves=game.moves({verbose:true});

 moves.sort((a,b)=>
  (b.captured?val({type:b.captured}):0)-
  (a.captured?val({type:a.captured}):0)
 );

 let best=-Infinity;

 for(const m of moves){
  game.move(m);

  let score=-search(
   depth-1,
   -beta,
   -alpha
  );

  game.undo();

  best=Math.max(best,score);
  alpha=Math.max(alpha,score);

  if(alpha>=beta)break;
 }

 return best===-Infinity?evaluate():best;
}

function chooseMove(){
 const depth=
  Number(document.getElementById('level').value)>=4
   ?3:2;

 const moves=game.moves({verbose:true});

 if(!moves.length)return null;

 let scored=moves.map(m=>{
  game.move(m);

  let score=-search(
   depth-1,
   -Infinity,
   Infinity
  );

  game.undo();

  return {m,score};
 });

 scored.sort((a,b)=>b.score-a.score);

 let pool=[scored[0]];

 if(document.getElementById('human').value==='yes'){
  const level=Number(
   document.getElementById('level').value
  );

  const count=
   level<=2?Math.min(5,scored.length):
   level===3?Math.min(3,scored.length):
   Math.min(2,scored.length);

  pool=scored.slice(0,count);

  const weights=pool.map((_,i)=>1/(i+1));

  let x=Math.random()*
   weights.reduce((a,b)=>a+b,0);

  let idx=0;

  for(;idx<weights.length-1;idx++){
   x-=weights[idx];
   if(x<=0)break;
  }

  return {
   chosen:pool[idx],
   best:scored[0],
   total:scored.length
  };
 }

 return {
  chosen:scored[0],
  best:scored[0],
  total:scored.length
 };
}

document.getElementById('hint').onclick=()=>{
 if(game.game_over()){
  statusEl.textContent='Oyun bitmiş. Yeni oyun başlat.';
  return;
 }

 const btn=document.getElementById('hint');

 btn.disabled=true;
 btn.textContent='Analiz ediliyor…';

 setTimeout(()=>{
  const result=chooseMove();

  btn.disabled=false;
  btn.textContent='✦ En iyi hamleyi öner';

  if(!result)return;

  const {chosen,best}=result;
  const m=chosen.m;
  const same=chosen===best;

  document.getElementById('advice').innerHTML=
   '<strong>Öneri: '+m.san+'</strong>'+
   '<div class="small">'+
   (same
    ?'Bu aramada en yüksek puanlı hamle.'
    :'İnsan gibi mod: En iyi hamle yerine güçlü bir alternatif seçildi.'
   )+
   '<br>Başlangıç: '+m.from+
   ' → Hedef: '+m.to+
   (m.captured?' · Taş alıyor':'')+
   '</div>';

  selected=m.from;

  legal=game.moves({
   square:m.from,
   verbose:true
  });

  render();

  statusEl.textContent=
   'Önerilen hedef kare tahtada işaretlendi. Hamleyi yapmak için hedef kareye dok.';
 },60);
};

render();

</script>
</body>
</html> Ai chess bot
