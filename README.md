
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Field Tracker</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}
.app{display:flex;flex-direction:column;height:100dvh;}

.header{padding:12px;background:#0f172a;}
input, textarea{
width:100%;
padding:12px;
border-radius:10px;
border:none;
margin-bottom:8px;
font-size:14px;
}

textarea{resize:none;height:60px;}

.status{font-size:13px;opacity:0.7;}
#map{flex:1;}

.controls{display:flex;gap:10px;padding:10px;background:#0f172a;}
button{flex:1;padding:16px;border:none;border-radius:12px;font-weight:bold;color:white;}
.in{background:#22c55e;}
.out{background:#ef4444;}

.popup-item{
padding:8px;
margin-bottom:6px;
background:#1f2937;
border-radius:8px;
font-size:13px;
}
</style>
</head>

<body>

<div class="app">

<div class="header">
<h3>Field Tracker</h3>

<input id="name" placeholder="Enter name">

<textarea id="purpose" placeholder="Enter purpose / activity (e.g. Site visit, inspection)"></textarea>

<div class="status" id="status">📡 Starting...</div>
</div>

<div id="map"></div>

<div class="controls">
<button class="in" onclick="save('IN')">TIME IN</button>
<button class="out" onclick="save('OUT')">TIME OUT</button>
</div>

</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// FIREBASE
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
let myMarker;

// GPS
navigator.geolocation.watchPosition(pos=>{
lat = pos.coords.latitude;
lon = pos.coords.longitude;

document.getElementById("status").innerText="📍 GPS Ready";

if(myMarker) map.removeLayer(myMarker);
myMarker = L.marker([lat,lon]).addTo(map);

map.setView([lat,lon],17);
});

// SAVE
async function save(type){

const name = document.getElementById("name").value;
const purpose = document.getElementById("purpose").value;

if(!name || !purpose || !lat){
alert("Complete name, purpose, and wait GPS");
return;
}

await db.collection("attendance").add({
name,
purpose,
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
