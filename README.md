
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Tracker</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
:root{
--bg:#0b1220;
--card:#0f172a;
--green:#22c55e;
--red:#ef4444;
--text:#fff;
--muted:#94a3b8;
}

body{
margin:0;
font-family:Arial;
background:var(--bg);
color:var(--text);
}

/* LAYOUT */
.container{
display:flex;
flex-direction:column;
height:100vh;
}

/* HEADER */
.header{
padding:12px;
background:var(--card);
}

input{
width:100%;
padding:12px;
border-radius:10px;
border:none;
margin-bottom:8px;
font-size:15px;
}

.status{
font-size:13px;
color:var(--muted);
}

/* MAP */
#map{
flex:1;
}

/* BUTTONS MOBILE */
.controls{
position:fixed;
bottom:0;
left:0;
right:0;
display:flex;
gap:10px;
padding:10px;
background:rgba(15,23,42,0.95);
}

button{
flex:1;
padding:16px;
border:none;
border-radius:12px;
font-size:15px;
font-weight:bold;
color:white;
}

.in{background:var(--green);}
.out{background:var(--red);}

/* DESKTOP MODE */
@media (min-width: 768px){

.container{
flex-direction:row;
}

.sidebar{
width:300px;
background:var(--card);
padding:15px;
}

#map{
height:100vh;
}

.controls{
position:static;
flex-direction:column;
padding:0;
margin-top:10px;
}

button{
width:100%;
}

}

</style>
</head>

<body>

<div class="container">

<!-- SIDEBAR / HEADER -->
<div class="header sidebar">

<h3>Field Tracker</h3>

<input id="name" placeholder="Enter name">

<div class="status" id="status">📡 Initializing...</div>

<div class="controls">
<button class="in" onclick="manualSave('IN')">TIME IN</button>
<button class="out" onclick="manualSave('OUT')">TIME OUT</button>
</div>

</div>

<!-- MAP -->
<div id="map"></div>

</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 FIREBASE CONFIG
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

// GPS
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

// SAVE
async function manualSave(type){

const name = document.getElementById("name").value;

if(!name || !lat){
alert("Missing name/GPS");
return;
}

await db.collection("attendance").add({
name,
type,
lat,
lon,
time:new Date().toLocaleTimeString(),
timestamp:Date.now()
});

alert(type + " saved");

}

</script>

</body>
</html>
