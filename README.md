<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
margin:0;
font-family:Arial;
background:#0b1220;
color:white;
}

.header{
padding:15px;
background:#0f172a;
}

h2{
margin:0 0 10px 0;
}

input{
width:100%;
padding:14px;
border-radius:12px;
border:none;
margin-bottom:10px;
font-size:16px;
}

.time{
font-size:14px;
margin-bottom:10px;
opacity:0.8;
}

.gps{
font-size:13px;
margin-bottom:10px;
}

.buttons{
display:flex;
gap:10px;
}

button{
flex:1;
padding:18px;
border:none;
border-radius:14px;
font-size:16px;
font-weight:bold;
color:white;
}

.in{
background:#22c55e;
}

.out{
background:#ef4444;
}

#map{
height:65vh;
}

.footer{
padding:10px;
text-align:center;
font-size:12px;
opacity:0.6;
}
</style>
</head>

<body>

<div class="header">
<h2>Field Attendance</h2>

<input id="name" placeholder="Enter your name">

<div class="time" id="time"></div>
<div class="gps" id="gps">📡 Checking GPS...</div>

<div class="buttons">
<button class="in" onclick="save('IN')">TIME IN</button>
<button class="out" onclick="save('OUT')">TIME OUT</button>
</div>
</div>

<div id="map"></div>

<div class="footer">
Company Field System
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 FIREBASE CONFIG (PASTE MO)
const firebaseConfig = {
apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.5,120.9],13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let currentLat = null;
let currentLon = null;

// 🕒 LIVE TIME
setInterval(()=>{
const now = new Date();
document.getElementById("time").innerText =
"🕒 " + now.toLocaleString();
},1000);

// 📡 GPS TRACK
navigator.geolocation.watchPosition(pos=>{

currentLat = pos.coords.latitude;
currentLon = pos.coords.longitude;

document.getElementById("gps").innerText =
"📍 GPS Ready";

map.setView([currentLat,currentLon],16);

L.marker([currentLat,currentLon]).addTo(map);

}, err=>{
document.getElementById("gps").innerText =
"❌ GPS Error - Enable Location";
});

// SAVE FUNCTION
async function save(type){

const name = document.getElementById("name").value;

if(!name){
alert("Enter your name");
return;
}

if(!currentLat){
alert("Waiting for GPS...");
return;
}

try{

const now = new Date();

await db.collection("attendance").add({
name,
type,
lat: currentLat,
lon: currentLon,
time: now.toLocaleTimeString(),
date: now.toLocaleDateString(),
timestamp: now.getTime()
});

alert("✅ " + type + " Saved");

}catch(e){
alert("❌ Error: " + e.message);
}

}

</script>

</body>
</html>
