
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Field Portal</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
margin:0;
font-family:system-ui,Arial;
background:#0b1220;
color:white;
overflow:hidden;
}

.app{
display:flex;
flex-direction:column;
height:100dvh;
}

/* HEADER */
.header{
padding:12px;
background:#0f172a;
box-shadow:0 2px 8px rgba(0,0,0,.3);
}

.title{
font-size:18px;
font-weight:700;
margin-bottom:10px;
}

input, textarea{
width:100%;
padding:12px;
border:none;
border-radius:10px;
margin-bottom:8px;
font-size:14px;
}

textarea{
resize:none;
height:60px;
}

.status{
font-size:13px;
opacity:.8;
}

/* MAP */
#map{
flex:1;
}

/* FOOTER BUTTONS */
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
font-size:15px;
font-weight:700;
color:white;
}

.in{background:#22c55e;}
.out{background:#ef4444;}

/* PIN */
.pin{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
border:1px solid #374151;
}

/* POPUP */
.popup-box{
max-height:250px;
overflow-y:auto;
}

.card{
background:#1f2937;
padding:10px;
border-radius:12px;
margin-bottom:8px;
}

.row{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:6px;
}

.tag{
font-size:11px;
padding:3px 8px;
border-radius:8px;
font-weight:bold;
}

.inTag{background:#22c55e;}
.outTag{background:#ef4444;}

.purpose{
padding:7px;
background:#111827;
border-left:3px solid #3b82f6;
border-radius:8px;
font-size:12px;
margin-top:6px;
}

.time{
font-size:12px;
opacity:.7;
}

</style>
</head>

<body>

<div class="app">

<div class="header">
<div class="title">Field Portal</div>

<input id="name" placeholder="Enter name">

<textarea id="purpose" placeholder="Enter purpose"></textarea>

<div class="status" id="status">📡 Waiting GPS...</div>
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
const db=firebase.firestore();

// MAP
const map=L.map('map').setView([15.5,120.9],15);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

let lat=null, lon=null;
let myMarker;
let markers=[];

// GPS
navigator.geolocation.watchPosition(pos=>{

lat=pos.coords.latitude;
lon=pos.coords.longitude;

document.getElementById("status").innerText="📍 GPS Ready";

if(myMarker) map.removeLayer(myMarker);

myMarker=L.marker([lat,lon]).addTo(map)
.bindPopup("📍 You are here");

map.setView([lat,lon],17);

},()=>{
document.getElementById("status").innerText="❌ Enable GPS";
});

// SAVE
async function save(type){

const name=document.getElementById("name").value.trim();
const purpose=document.getElementById("purpose").value.trim();

if(!name || !purpose || !lat){
alert("Complete name, purpose and GPS");
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

alert(type+" saved");

}

// LIVE PINS
db.collection("attendance").orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped={};

snapshot.forEach(doc=>{

let d=doc.data();
let key=d.lat.toFixed(5)+","+d.lon.toFixed(5);

if(!grouped[key]) grouped[key]=[];
grouped[key].push(d);

});

// CREATE MARKERS
Object.keys(grouped).forEach(key=>{

let logs=grouped[key];
let lat=logs[0].lat;
let lon=logs[0].lon;

logs.sort((a,b)=>b.timestamp-a.timestamp);

let html=`<div class="popup-box">`;

logs.forEach(l=>{

let tag=l.type==="IN"?"inTag":"outTag";

html+=`
<div class="card">

<div class="row">
<b>${l.name}</b>
<div class="tag ${tag}">${l.type}</div>
</div>

<div class="time">${l.time}</div>

<div class="purpose">
📌 ${l.purpose || "No purpose"}
</div>

</div>
`;

});

html+=`</div>`;

let icon=L.divIcon({
html:`<div class="pin">📍 ${logs.length}</div>`,
className:"",
iconSize:[60,30]
});

let marker=L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

});

</script>

</body>
</html>
