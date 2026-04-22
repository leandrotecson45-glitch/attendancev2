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

input{
width:100%;
padding:14px;
border-radius:10px;
border:none;
margin-bottom:10px;
font-size:16px;
}

.buttons{
display:flex;
gap:10px;
}

button{
flex:1;
padding:15px;
border:none;
border-radius:12px;
font-size:16px;
font-weight:bold;
color:white;
}

.in{background:#22c55e;}
.out{background:#ef4444;}

#map{
height:70vh;
}
</style>
</head>

<body>

<div class="header">
<input id="name" placeholder="Employee Name">

<div class="buttons">
<button class="in" onclick="save('IN')">TIME IN</button>
<button class="out" onclick="save('OUT')">TIME OUT</button>
</div>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 PALITAN MO ITO
const firebaseConfig = {
apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png')
.addTo(map);

// 🔥 SAVE FUNCTION (SIMPLE + SURE WORK)
function save(type){

const name = document.getElementById("name").value;

if(!name){
alert("⚠️ Enter name");
return;
}

// GPS
navigator.geolocation.getCurrentPosition(async pos=>{

try{

const lat = pos.coords.latitude;
const lon = pos.coords.longitude;

const now = new Date();

await db.collection("attendance").add({
name: name,
type: type,
lat: lat,
lon: lon,
time: now.toLocaleTimeString(),
date: now.toLocaleDateString(),
timestamp: now.getTime()
});

// MAP PIN
L.marker([lat,lon]).addTo(map)
.bindPopup(`
<b>${name}</b><br>
${type}<br>
${now.toLocaleTimeString()}
`);

map.setView([lat,lon],17);

alert("✅ SUCCESS TIME " + type);

}catch(e){
console.error(e);
alert("❌ FIREBASE ERROR");
}

}, err=>{
alert("❌ GPS ERROR - enable location");
});

}

</script>

</body>
</html>
