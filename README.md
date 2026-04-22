
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Field Attendance</title>

<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#0f172a">

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
margin:0;
font-family:Arial;
background:#0b1220;
color:white;
}

/* HEADER */
.header{
padding:12px;
display:flex;
flex-direction:column;
gap:10px;
background:#0f172a;
}

/* INPUT STYLE */
input{
padding:14px;
border-radius:12px;
border:none;
font-size:16px;
outline:none;
}

/* BUTTONS */
.btns{
display:flex;
gap:10px;
}

button{
flex:1;
padding:16px;
border:none;
border-radius:14px;
font-size:16px;
font-weight:bold;
color:white;
}

.in{background:#22c55e;}
.out{background:#ef4444;}

/* MAP FULL SCREEN */
#map{
height:75vh;
border-top-left-radius:20px;
border-top-right-radius:20px;
overflow:hidden;
}

/* STATUS */
.status{
text-align:center;
font-size:12px;
opacity:0.7;
}

</style>
</head>

<body>

<div class="header">

<div class="status">📍 Field Attendance System</div>

<input id="name" placeholder="Enter your name">

<div class="btns">
<button class="in" onclick="save('TIME IN')">TIME IN</button>
<button class="out" onclick="save('TIME OUT')">TIME OUT</button>
</div>

</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// FIREBASE (same)
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
if(!name){
alert("⚠️ Enter name");
return;
}

gps((lat,lon)=>{

const data={
name,
type,
time:new Date().toLocaleString(),
lat,lon
};

// FIREBASE SAVE
db.collection("attendance").add(data);

// UI PIN
let color = type==="TIME IN" ? "#22c55e" : "#ef4444";

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

</script>

</body>
</html>
