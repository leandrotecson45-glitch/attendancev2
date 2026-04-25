<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Field Portal</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial, sans-serif;
}

body{
background:#0f172a;
color:#fff;
}

header{
padding:15px;
text-align:center;
font-size:22px;
font-weight:bold;
background:#111827;
}

.container{
padding:15px;
}

.card{
background:#1e293b;
padding:15px;
border-radius:14px;
margin-bottom:15px;
box-shadow:0 4px 10px rgba(0,0,0,.25);
}

label{
display:block;
margin-bottom:6px;
font-size:14px;
font-weight:bold;
}

select,input,textarea{
width:100%;
padding:12px;
border:none;
border-radius:10px;
outline:none;
font-size:14px;
margin-bottom:12px;
}

textarea{
resize:none;
height:70px;
}

button{
width:100%;
padding:14px;
border:none;
border-radius:10px;
font-size:15px;
font-weight:bold;
color:#fff;
cursor:pointer;
margin-bottom:10px;
}

.btn-in{
background:#16a34a;
}

.btn-out{
background:#dc2626;
}

#map{
height:420px;
border-radius:14px;
overflow:hidden;
margin-top:10px;
}

.status{
font-size:13px;
color:#cbd5e1;
margin-top:5px;
}

.pin-box{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:#fff;
border:1px solid #334155;
}

.popup-card{
background:#111827;
padding:10px;
border-radius:10px;
margin-bottom:8px;
color:#fff;
font-size:13px;
}

.tag{
display:inline-block;
padding:4px 8px;
border-radius:8px;
font-size:11px;
font-weight:bold;
margin-top:6px;
}

.in{
background:#16a34a;
}

.out{
background:#dc2626;
}

.purpose{
margin-top:8px;
padding:8px;
background:#1e293b;
border-left:4px solid #38bdf8;
border-radius:8px;
font-size:12px;
}
</style>
</head>
<body>

<header>📍 Field Portal</header>

<div class="container">

<div class="card">

<label>Select Employee</label>
<select id="name">
<option value="">Choose Employee</option>
<option>Juan Dela Cruz</option>
<option>Pedro Santos</option>
<option>Maria Cruz</option>
<option>Leandro Tecson</option>
<option>Supervisor 1</option>
</select>

<label>Purpose</label>
<textarea id="purpose" placeholder="Enter purpose here..."></textarea>

<button class="btn-in" onclick="saveLog('IN')">🟢 TIME IN</button>
<button class="btn-out" onclick="saveLog('OUT')">🔴 TIME OUT</button>

<div class="status" id="status">Waiting for GPS...</div>

</div>

<div id="map"></div>

</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// FIREBASE CONFIG
const firebaseConfig = {
 apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.486,120.967],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
maxZoom:19
}).addTo(map);

let userLat = null;
let userLng = null;
let myMarker = null;
let markers = [];

// GPS WATCH
navigator.geolocation.watchPosition(pos=>{

userLat = pos.coords.latitude;
userLng = pos.coords.longitude;

document.getElementById("status").innerText =
"GPS Ready";

if(myMarker){
map.removeLayer(myMarker);
}

myMarker = L.marker([userLat,userLng]).addTo(map)
.bindPopup("You are here");

map.setView([userLat,userLng],16);

},err=>{
document.getElementById("status").innerText =
"Enable GPS Permission";
});

// SAVE LOG
function saveLog(type){

const name = document.getElementById("name").value;
const purpose = document.getElementById("purpose").value.trim();

if(!name){
alert("Select employee name");
return;
}

if(!purpose){
alert("Enter purpose");
return;
}

if(userLat===null){
alert("Waiting for GPS");
return;
}

db.collection("attendance").add({
name:name,
purpose:purpose,
type:type,
lat:userLat,
lon:userLng,
time:new Date().toLocaleTimeString(),
timestamp:Date.now()
})
.then(()=>{
alert(type + " saved");
document.getElementById("purpose").value="";
})
.catch(err=>{
alert("Error saving");
});

}

// LIVE MARKERS
db.collection("attendance")
.orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped = {};

snapshot.forEach(doc=>{

let d = doc.data();

let key =
d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]){
grouped[key]=[];
}

grouped[key].push(d);

});

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];

let lat = logs[0].lat;
let lon = logs[0].lon;

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html = "";

logs.forEach(l=>{

html += `
<div class="popup-card">
<b>${l.name}</b><br>
${l.time}<br>
<span class="tag ${l.type==='IN'?'in':'out'}">${l.type}</span>
<div class="purpose">📌 ${l.purpose}</div>
</div>
`;

});

let icon = L.divIcon({
html:`<div class="pin-box">📍 ${logs.length}</div>`,
className:"",
iconSize:[60,30]
});

let marker = L.marker([lat,lon],{icon:icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

});

</script>
</body>
</html>
