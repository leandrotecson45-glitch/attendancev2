
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Field Tracker</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}

.app{display:flex;flex-direction:column;height:100dvh;}

.header{
padding:12px;
background:#0f172a;
}

input{
width:100%;
padding:12px;
border-radius:10px;
border:none;
margin-bottom:8px;
}

.status{font-size:13px;opacity:0.7;}

#map{flex:1;}

.controls{
display:flex;
gap:10px;
padding:10px;
background:#0f172a;
}

button{
flex:1;
padding:16px;
border:none;
border-radius:12px;
font-weight:bold;
color:white;
}

.in{background:#22c55e;}
.out{background:#ef4444;}

/* 🔥 POPUP DESIGN */
.popup-box{
font-size:13px;
max-height:180px;
overflow-y:auto;
}

.popup-header{
font-weight:bold;
margin-bottom:6px;
}

.popup-item{
padding:8px;
margin-bottom:6px;
background:#1f2937;
border-radius:8px;
display:flex;
justify-content:space-between;
align-items:center;
}

.tag{
font-size:11px;
padding:2px 6px;
border-radius:6px;
}

.inTag{background:#22c55e;}
.outTag{background:#ef4444;}
.autoTag{background:#3b82f6;}

</style>
</head>

<body>

<div class="app">

<div class="header">
<h3>Field Tracker</h3>
<input id="name" placeholder="Enter name">
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
let markers=[];

// GPS
navigator.geolocation.watchPosition(pos=>{
lat = pos.coords.latitude;
lon = pos.coords.longitude;

document.getElementById("status").innerText="📍 GPS Ready";

if(myMarker) map.removeLayer(myMarker);
myMarker = L.marker([lat,lon]).addTo(map);

map.setView([lat,lon],17);
});

// 🔥 REALTIME WITH BEAUTIFUL POPUP
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped = {};

// GROUP BY LOCATION
snapshot.forEach(doc=>{
let d = doc.data();
let key = d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);
});

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs = grouped[key];
let lat = logs[0].lat;
let lon = logs[0].lon;

// 🔥 POPUP UI
let html = `<div class="popup-box">`;
html += `<div class="popup-header">📍 ${logs.length} Records</div>`;

logs.forEach(l=>{

let tagClass = "autoTag";
if(l.type==="IN") tagClass="inTag";
if(l.type==="OUT") tagClass="outTag";

html += `
<div class="popup-item">
<div>
<b>${l.name}</b><br>
<small>${l.time}</small>
</div>
<div class="tag ${tagClass}">
${l.type}
</div>
</div>
`;

});

html += `</div>`;

// LABEL
let iconHTML = `
<div style="
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
color:white;">
📍 ${logs.length}
</div>
`;

let icon = L.divIcon({
html:iconHTML,
className:"",
iconSize:[60,30]
});

let marker = L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

});

// SAVE
async function save(type){

const name=document.getElementById("name").value;

if(!name || !lat){
alert("Enter name / wait GPS");
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

alert(type+" saved");

}

</script>

</body>
</html>
