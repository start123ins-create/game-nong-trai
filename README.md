
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Nông Trại Vui Vẻ</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;
  font-family:Arial,sans-serif;
  background:#b8e994;
  color:#3b2f20;
  text-align:center;
}
header{
  background:#388e3c;
  color:white;
  padding:18px 10px;
}
h1{margin:0 0 10px}
.stats{
  display:flex;
  justify-content:center;
  gap:18px;
  flex-wrap:wrap;
  font-size:18px;
}
main{padding:15px}
.tools{
  display:flex;
  justify-content:center;
  flex-wrap:wrap;
  gap:8px;
  margin-bottom:18px;
}
button{
  border:0;
  border-radius:12px;
  padding:12px 15px;
  background:#fff3cd;
  font-weight:bold;
  font-size:15px;
  cursor:pointer;
}
button.active{background:#ffca28}
button:active{transform:scale(.96)}
.farm{
  display:grid;
  grid-template-columns:repeat(4,minmax(55px,100px));
  justify-content:center;
  gap:8px;
  max-width:440px;
  margin:auto;
  padding:12px;
  background:#69b34c;
  border:5px solid #8b5a2b;
  border-radius:16px;
}
.plot{
  aspect-ratio:1;
  background:#986239;
  border:3px solid #70451f;
  border-radius:10px;
  font-size:36px;
  display:flex;
  align-items:center;
  justify-content:center;
  flex-direction:column;
}
.plot small{font-size:11px;color:white}
#message{
  margin:16px auto;
  min-height:25px;
  font-weight:bold;
}
footer{padding:12px;font-size:13px}
</style>
</head>
<body>
<header>
  <h1>🌻 NÔNG TRẠI VUI VẺ 🌻</h1>
  <div class="stats">
    <span>💰 Xu: <b id="coins">100</b></span>
    <span>🥕 Cà rốt: <b id="carrots">0</b></span>
    <span>⭐ Cấp: <b id="level">1</b></span>
  </div>
</header>

<main>
  <h3>Chọn công cụ rồi nhấn vào ô đất!</h3>
  <div class="tools">
    <button class="active" onclick="choose('plant',this)">🌱 Gieo hạt (10 xu)</button>
    <button onclick="choose('water',this)">💧 Tưới nước</button>
    <button onclick="choose('harvest',this)">🥕 Thu hoạch</button>
    <button onclick="sell()">🛒 Bán cà rốt (+20 xu)</button>
  </div>

  <div class="farm" id="farm"></div>
  <div id="message">Chào mừng bạn đến với nông trại!</div>
  <button onclick="resetGame()">🔄 Chơi lại</button>
</main>

<footer>Chăm sóc cây trồng để trở thành nông dân giàu có!</footer>

<script>
let coins = 100;
let carrots = 0;
let tool = "plant";
let plots = Array.from({length:16}, () => ({
  state:"empty",
  time:0
}));

function choose(name,button){
  tool=name;
  document.querySelectorAll(".tools button")
    .forEach(b=>b.classList.remove("active"));
  button.classList.add("active");
  say("Đã chọn công cụ: "+button.textContent);
}

function say(text){
  document.getElementById("message").textContent=text;
}

function render(){
  document.getElementById("coins").textContent=coins;
  document.getElementById("carrots").textContent=carrots;
  document.getElementById("level").textContent=
    Math.floor(carrots/10)+1;

  const farm=document.getElementById("farm");
  farm.innerHTML="";

  plots.forEach((p,i)=>{
    const div=document.createElement("button");
    div.className="plot";
    div.style.width="100%";

    let emoji="🟫";
    let label="Đất trống";

    if(p.state==="seed"){
      emoji="🌱"; label="Mới gieo";
    }else if(p.state==="growing"){
      emoji="🌿"; label="Đang lớn";
    }else if(p.state==="ready"){
      emoji="🥕"; label="Chín rồi!";
    }

    div.innerHTML=emoji+"<small>"+label+"</small>";
    div.onclick=()=>act(i);
    farm.appendChild(div);
  });
}

function act(i){
  const p=plots[i];

  if(tool==="plant"){
    if(p.state!=="empty"){
      say("Ô đất này đã có cây rồi!");
      return;
    }
    if(coins<10){
      say("Bạn không đủ 10 xu để mua hạt giống!");
      return;
    }
    coins-=10;
    p.state="seed";
    p.time=Date.now();
    say("Bạn đã gieo hạt! Hãy tưới nước nhé.");
  }

  else if(tool==="water"){
    if(p.state==="seed"){
      p.state="growing";
      p.time=Date.now();
      say("Tưới nước thành công! Cây đang lớn.");
    }else if(p.state==="growing"){
      say("Cây đang phát triển, hãy chờ một chút!");
    }else{
      say("Ô đất này chưa cần tưới nước.");
    }
  }

  else if(tool==="harvest"){
    if(p.state==="ready"){
      carrots++;
      p.state="empty";
      p.time=0;
      say("Thu hoạch thành công! Bạn nhận được 1 cà rốt.");
    }else{
      say("Cà rốt chưa chín! Hãy chăm sóc cây.");
    }
  }

  render();
}

function sell(){
  if(carrots<=0){
    say("Bạn chưa có cà rốt để bán!");
    return;
  }
  coins+=carrots*20;
  say("Đã bán "+carrots+" củ cà rốt!");
  carrots=0;
  render();
}

function resetGame(){
  if(!confirm("Bạn có chắc muốn chơi lại từ đầu?")) return;
  coins=100;
  carrots=0;
  plots=Array.from({length:16},()=>({
    state:"empty",time:0
  }));
  render();
  say("Nông trại mới đã sẵn sàng!");
}

// Cây phát triển sau 10 giây kể từ khi tưới nước
setInterval(()=>{
  let changed=false;
  plots.forEach(p=>{
    if(p.state==="growing" &&
       Date.now()-p.time>=10000){
      p.state="ready";
      changed=true;
    }
  });
  if(changed){
    render();
    say("🥕 Có cà rốt đã chín! Hãy thu hoạch.");
  }
},1000);

render();
</script>
</body>
</html>
