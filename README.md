
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Tracker</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}
.header{padding:12px;background:#0f172a;}
input{width:100%;padding:12px;border-radius:10px;border:none;margin-bottom:8px;}
.status{font-size:13px;color:#94a3b8;}
#map{height:60vh;}
.controls{position:fixed;bottom:0;width:100%;display:flex;gap:10px;padding:10px;background:#0f172a;}
button{flex:1;padding:16px;border:none;border-radius:12px;color:white;font-weight:bold;}
.in{background:#22c55e;}
.out{background:#ef4444;}
</style>
</head>

<body>

<div class="header">
<input id="name" placeholder="Enter name">
<div class="status" id="status">Initializing...</div>
</div>

<div id="map"></div>

<div class="controls">
<button class="in" onclick="manualSave('IN')">TIME IN</button>
<button class="out" onclick="manualSave('OUT')">TIME OUT</button>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 FIREBASE
const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.5,120.9],15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let lat=null, lon=null;
let marker;

// 📡 GPS WATCH
navigator.geolocation.watchPosition(pos=>{

lat = pos.coords.latitude;
lon = pos.coords.longitude;

document.getElementById("status").innerText =
"📍 GPS Ready";

if(marker) map.removeLayer(marker);
marker = L.marker([lat,lon]).addTo(map);

map.setView([lat,lon],17);

},()=>{
document.getElementById("status").innerText =
"❌ Enable GPS";
});

// 💾 OFFLINE STORAGE
function saveLocal(data){
let logs = JSON.parse(localStorage.getItem("offline") || "[]");
logs.push(data);
localStorage.setItem("offline", JSON.stringify(logs));
}

// 🔄 SYNC FUNCTION
async function syncData(){

let logs = JSON.parse(localStorage.getItem("offline") || "[]");

if(logs.length === 0) return;

document.getElementById("status").innerText =
"🔄 Syncing " + logs.length;

for(let d of logs){
await db.collection("attendance").add(d);
}

localStorage.removeItem("offline");

document.getElementById("status").innerText =
"✅ Synced";

}

// 🌐 CHECK INTERNET
setInterval(()=>{
if(navigator.onLine){
syncData();
}
},5000);

// 🟢 AUTO TRACK EVERY 30s
setInterval(()=>{

if(!lat) return;

const name = document.getElementById("name").value;
if(!name) return;

let data = {
name,
type:"AUTO",
lat,
lon,
time:new Date().toLocaleTimeString(),
timestamp:Date.now()
};

// SAVE ONLINE OR OFFLINE
if(navigator.onLine){
db.collection("attendance").add(data);
}else{
saveLocal(data);
}

},30000);

// 🧑‍💼 MANUAL
async function manualSave(type){

const name = document.getElementById("name").value;

if(!name || !lat){
alert("Missing name/GPS");
return;
}

let data = {
name,
type,
lat,
lon,
time:new Date().toLocaleTimeString(),
timestamp:Date.now()
};

if(navigator.onLine){
await db.collection("attendance").add(data);
}else{
saveLocal(data);
}

alert("Saved");

}

</script>

</body>
</html>
