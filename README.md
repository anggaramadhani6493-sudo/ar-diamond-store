<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ANGGA STORE</title>

<script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>

<style>
body { margin:0; font-family:Arial; background:#0f172a; color:white; }

.header {
  background:linear-gradient(90deg,#22c55e,#16a34a);
  padding:15px;
  text-align:center;
  font-size:22px;
  font-weight:bold;
}

.grid {
  display:grid;
  grid-template-columns:repeat(2,1fr);
  gap:15px;
  padding:15px;
}

.card {
  background:#1e293b;
  padding:20px;
  border-radius:12px;
  text-align:center;
  cursor:pointer;
}

.form { display:none; padding:15px; }

input {
  width:100%;
  padding:10px;
  border-radius:8px;
  border:none;
  margin-bottom:10px;
}

.nominal-item {
  background:#1e293b;
  padding:10px;
  border-radius:8px;
  margin:5px 0;
  cursor:pointer;
}

.selected { background:#22c55e; color:black; }

button {
  width:100%;
  padding:10px;
  margin-top:8px;
  border:none;
  border-radius:8px;
  cursor:pointer;
}

.wa { background:#25D366; }
.qr { background:#3b82f6; }

.qris {
  display:none;
  text-align:center;
  margin-top:10px;
  background:#1e293b;
  padding:10px;
  border-radius:10px;
}

.status {
  margin-top:10px;
  background:#1e293b;
  padding:10px;
  border-radius:10px;
}
</style>
</head>

<body>

<div class="header">
🔥 ANGGA STORE
<br>
<button onclick="loginAdmin()">🔐 Admin</button>
</div>

<div class="grid" id="gameList">
<div class="card" onclick="pilihGame('Free Fire')">🔥 Free Fire</div>
<div class="card" onclick="pilihGame('Mobile Legends')">⚔️ Mobile Legends</div>
<div class="card" onclick="pilihGame('PUBG Mobile')">🎯 PUBG Mobile</div>
<div class="card" onclick="pilihGame('Valorant')">💥 Valorant</div>
</div>

<div class="form" id="form">

<button onclick="kembali()">⬅️ Kembali</button>
<h3 id="judul"></h3>

<input type="text" id="userId" placeholder="Masukkan ID Game">
<input type="text" id="nickname" placeholder="Nickname (opsional)">

<div id="listNominal"></div>

<button class="wa" onclick="beliWA()">WhatsApp</button>
<button class="qr" onclick="tampilQR()">QRIS</button>

<div class="qris" id="qrisBox">
<h3>💳 QRIS Pembayaran</h3>
<div id="qrcode"></div>
<br>
<div id="nominalBox"></div>

<input type="file" id="bukti">
<button onclick="kirim()">Upload Bukti</button>
</div>

<div class="status" id="statusBox"></div>

</div>

<script>

let data = JSON.parse(localStorage.getItem("harga")) || {
"Free Fire":["70 DM - 10000","100 DM - 13000"],
"Mobile Legends":["44 DM - 12000","59 DM - 16000"],
"PUBG Mobile":["60 UC - 15000","120 UC - 28000"],
"Valorant":["125 VP - 15000","420 VP - 50000"]
};

let game = "";
let nominal = "";
let harga = "";

function pilihGame(nama){
  game = nama;
  document.getElementById("gameList").style.display="none";
  document.getElementById("form").style.display="block";
  document.getElementById("judul").innerText = nama;

  let box = document.getElementById("listNominal");
  box.innerHTML = "";

  data[nama].forEach(item=>{
    let div = document.createElement("div");
    div.className = "nominal-item";
    div.innerText = item;

    div.onclick = () => {
      document.querySelectorAll(".nominal-item")
        .forEach(x=>x.classList.remove("selected"));

      div.classList.add("selected");

      nominal = item;
      harga = item.split("-")[1].trim();
    };

    box.appendChild(div);
  });
}

function kembali(){
  location.reload();
}

function beliWA(){
  let id = document.getElementById("userId").value;
  let nick = document.getElementById("nickname").value;

  if(!id || !nominal) return alert("Lengkapi data!");

  window.open(`https://wa.me/62895352027953?text=TopUp ${game}%0AID:${id}%0ANick:${nick}%0A${nominal}`);
}

function tampilQR(){
  let id = document.getElementById("userId").value;
  if(!id || !nominal) return alert("Isi dulu!");

  document.getElementById("qrisBox").style.display="block";
  document.getElementById("nominalBox").innerText="Bayar: Rp " + harga;

  document.getElementById("qrcode").innerHTML="";
  new QRCode(document.getElementById("qrcode"), {
    text: "PAYMENT|" + harga + "|" + game + "|" + id,
    width: 180,
    height: 180
  });
}

function kirim(){
  let id = document.getElementById("userId").value;
  let file = document.getElementById("bukti").files[0];

  if(!file) return alert("Upload bukti!");

  let reader = new FileReader();

  reader.onload = function(e){
    let orders = JSON.parse(localStorage.getItem("orders")) || [];

    orders.push({
      game,
      id,
      nominal,
      status:"Pending",
      bukti:e.target.result
    });

    localStorage.setItem("orders", JSON.stringify(orders));

    alert("Order masuk!");
    tampilStatus();
  };

  reader.readAsDataURL(file);
}

function tampilStatus(){
  let orders = JSON.parse(localStorage.getItem("orders")) || [];
  let box = document.getElementById("statusBox");

  box.innerHTML = "<b>Status:</b><br>";

  orders.forEach(o=>{
    box.innerHTML += `
    ${o.game}<br>
    ID:${o.id}<br>
    ${o.nominal}<br>
    Status:${o.status}<br>
    <hr>
    `;
  });
}

function loginAdmin(){
  let p = prompt("Password?");
  if(p !== "admin123") return alert("Salah!");
  alert("Admin Mode aktif");
}

window.onload = tampilStatus;

</script>

</body>
</html>
