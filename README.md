<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance App</title>

<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#0f172a">

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;}
.top{display:flex;padding:10px;gap:5px;background:#0f172a;}
input{flex:1;padding:10px;}
button{padding:10px;border:none;color:white;}
.in{background:#22c55e;}
.out{background:#ef4444;}
#map{height:90vh;}
</style>
</head>

<body>

<div class="top">
<input id="name" placeholder="Name">
<button class="in" onclick="save('TIME IN')">IN</button>
<button class="out" onclick="save('TIME OUT')">OUT</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 FIREBASE CONFIG (PALITAN MO)
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

// GPS
function gps(cb){
navigator.geolocation.getCurrentPosition(p=>{
cb(p.coords.latitude,p.coords.longitude);
});
}

// SAVE
function save(type){

const name=document.getElementById("name").value;
if(!name) return alert("Enter name");

gps((lat,lon)=>{

const data={
name,
type,
time:new Date().toLocaleString(),
lat,lon
};

// FIREBASE SAVE (NO OVERWRITE)
db.collection("attendance").add(data);

let color = type==="TIME IN" ? "blue" : "red";

L.circleMarker([lat,lon],{
radius:10,
color:color,
fillColor:color,
fillOpacity:0.9
}).addTo(map)
.bindPopup(`
<b>${name}</b><br>
${type}<br>
${data.time}
`);

map.setView([lat,lon],17);

});

}

// 🔥 SERVICE WORKER REGISTER
if('serviceWorker' in navigator){
navigator.serviceWorker.register('sw.js');
}

</script>

</body>
</html>
