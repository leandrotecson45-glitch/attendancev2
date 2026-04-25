
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,viewport-fit=cover">
<title>Field Portal</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

html,body{
height:100%;
overflow:hidden;
background:#0f172a;
color:#fff;
}

/* APP */
.app{
height:100dvh;
display:flex;
flex-direction:column;
}

/* HEADER */
header{
padding:14px;
text-align:center;
font-size:20px;
font-weight:700;
background:#111827;
box-shadow:0 4px 12px rgba(0,0,0,.25);
z-index:5;
}

/* FORM PANEL */
.panel{
padding:12px;
background:#1e293b;
display:flex;
flex-direction:column;
gap:10px;
z-index:5;
}

select,
textarea{
width:100%;
padding:12px;
border:none;
border-radius:12px;
font-size:16px;
outline:none;
}

textarea{
resize:none;
height:74px;
}

/* BUTTONS */
.row{
display:flex;
gap:10px;
}

button{
flex:1;
padding:14px;
border:none;
border-radius:12px;
font-size:15px;
font-weight:700;
color:#fff;
}

.in-btn{
background:#16a34a;
}

.out-btn{
background:#dc2626;
}

/* STATUS */
.status{
font-size:13px;
color:#cbd5e1;
}

/* MAP */
#map{
flex:1;
width:100%;
}

/* PINS */
.pin-box{
background:#111827;
padding:6px 10px;
border-radius:20px;
font-size:12px;
font-weight:bold;
color:#fff;
border:1px solid #334155;
}

/* POPUP */
.leaflet-popup-content-wrapper{
background:#0f172a;
color:#fff;
border-radius:16px;
}

.leaflet-popup-tip{
background:#0f172a;
}

.leaflet-popup-content{
margin:10px;
max-height:260px;
overflow-y:auto;
width:260px !important;
}

.popup-card{
background:#111827;
padding:10px;
border-radius:12px;
margin-bottom:8px;
}

.top{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:8px;
}

.tag{
padding:4px 8px;
border-radius:999px;
font-size:11px;
font-weight:700;
}

.tag-in{
background:#14532d;
color:#86efac;
}

.tag-out{
background:#7f1d1d;
color:#fca5a5;
}

.time{
font-size:11px;
color:#94a3b8;
margin-top:3px;
}

.purpose{
margin-top:8px;
padding:8px;
border-left:4px solid #38bdf8;
background:#0f172a;
border-radius:10px;
font-size:12px;
}

/* MOBILE SAFE SPACE */
@media(max-width:600px){
header{
font-size:18px;
padding:12px;
}

select,
textarea{
font-size:16px;
}

button{
font-size:14px;
padding:13px;
}
}
</style>
</head>
<body>

<div class="app">

<header>📍 Field Portal</header>

<div class="panel">

<select id="name">
<option value="">Select Employee</option>
<option>Mark Gabriel Marcelo</option>
<option>Mariel Macapulay</option>
<option>Allen Rodolfo</option>
<option>Mark Tolentino</option>
<option>Bego Patricio</option>
  <option>Mark Sabatin</option>
    <option>Gian Carlo Claus</option>
    <option>Arjay Hulinganga</option>
    <option>June Joven</option>
    <option>Ken Tintero</option>
    <option>Michael Cababag</option>
    <option>Paul Navarrete</option>
    <option>Angelique Lei Balete</option>
    <option>June Agatin</option>
   <option>Rodrigo Villota</option>
</select>

<textarea
id="purpose"
placeholder="Enter purpose here..."
></textarea>

<div class="row">
<button class="in-btn" onclick="saveLog('IN')">TIME IN</button>
<button class="out-btn" onclick="saveLog('OUT')">TIME OUT</button>
</div>

<div class="status" id="status">
Waiting GPS...
</div>

</div>

<div id="map"></div>

</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
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
const map = L.map("map").setView([15.486,120.967],13);

L.tileLayer(
"https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
{maxZoom:19}
).addTo(map);

let userLat=null;
let userLng=null;
let myMarker=null;
let markers=[];

// =====================================
// AUTO ZOOM WHEN TYPING (PHONE KEYBOARD)
// =====================================
const purposeInput =
document.getElementById("purpose");

const nameInput =
document.getElementById("name");

function focusMode(){

document.querySelector(".panel")
.scrollIntoView({
behavior:"smooth",
block:"start"
});

setTimeout(()=>{
map.invalidateSize();
},400);

}

function normalMode(){

setTimeout(()=>{
map.invalidateSize();
},500);

}

purposeInput.addEventListener("focus",focusMode);
nameInput.addEventListener("focus",focusMode);

purposeInput.addEventListener("blur",normalMode);
nameInput.addEventListener("blur",normalMode);

// =====================================
// GPS
// =====================================
navigator.geolocation.watchPosition(pos=>{

userLat = pos.coords.latitude;
userLng = pos.coords.longitude;

document.getElementById("status")
.innerText="📍 GPS Ready";

if(myMarker){
map.removeLayer(myMarker);
}

myMarker =
L.marker([userLat,userLng])
.addTo(map)
.bindPopup("You are here");

map.setView([userLat,userLng],16);

},err=>{

document.getElementById("status")
.innerText="Enable GPS Permission";

});

// =====================================
// SAVE
// =====================================
function saveLog(type){

const name =
document.getElementById("name").value;

const purpose =
document.getElementById("purpose")
.value.trim();

if(!name){
alert("Select employee");
return;
}

if(!purpose){
alert("Enter purpose");
return;
}

if(userLat===null){
alert("Waiting GPS");
return;
}

db.collection("attendance")
.add({
name:name,
purpose:purpose,
type:type,
lat:userLat,
lon:userLng,
time:new Date().toLocaleTimeString(),
timestamp:Date.now()
})
.then(()=>{

alert(type+" saved");

document.getElementById("purpose").value="";

})
.catch(()=>{

alert("Save failed");

});

}

// =====================================
// LIVE PINS
// =====================================
db.collection("attendance")
.orderBy("timestamp")
.onSnapshot(snapshot=>{

markers.forEach(m=>map.removeLayer(m));
markers=[];

let grouped={};

snapshot.forEach(doc=>{

let d=doc.data();

let key =
d.lat.toFixed(5)+","+
d.lon.toFixed(5);

if(!grouped[key]){
grouped[key]=[];
}

grouped[key].push(d);

});

Object.keys(grouped).forEach(key=>{

let logs=grouped[key];

let lat=logs[0].lat;
let lon=logs[0].lon;

logs.sort((a,b)=>
b.timestamp-a.timestamp
);

let html="";

logs.forEach(l=>{

html += `
<div class="popup-card">

<div class="top">

<div>
<b>${l.name}</b>
<div class="time">
${l.time}
</div>
</div>

<div class="tag ${l.type==='IN'?'tag-in':'tag-out'}">
${l.type}
</div>

</div>

<div class="purpose">
📌 ${l.purpose}
</div>

</div>
`;

});

let icon=L.divIcon({
html:`<div class="pin-box">📍 ${logs.length}</div>`,
className:"",
iconSize:[60,30]
});

let marker=
L.marker([lat,lon],{icon})
.addTo(map)
.bindPopup(html);

markers.push(marker);

});

});

</script>
</body>
</html>
