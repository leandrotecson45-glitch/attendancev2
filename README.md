<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field App</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}
.header{padding:10px;background:#0f172a;}
input{width:100%;padding:12px;margin-bottom:8px;border-radius:10px;border:none;}
.buttons{display:flex;gap:5px;}
button{flex:1;padding:12px;border:none;border-radius:10px;color:white;font-weight:bold;}
.in{background:#22c55e;}
.out{background:#ef4444;}
#map{height:60vh;}
</style>
</head>

<body>

<div class="header">
<input id="name" placeholder="Employee Name">
<input type="file" id="photo" accept="image/*" capture="environment">

<div class="buttons">
<button class="in" onclick="save('IN')">TIME IN</button>
<button class="out" onclick="save('OUT')">TIME OUT</button>
</div>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>

<script>

// 🔥 PASTE MO TAMANG CONFIG
const firebaseConfig = {
apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2",
  storageBucket: "attendance1-697b2.firebasestorage.app"
};

firebase.initializeApp(firebaseConfig);

const db = firebase.firestore();
const storage = firebase.storage();

// MAP
const map = L.map('map').setView([15.5,120.9],13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// DISTANCE
function distance(lat1, lon1, lat2, lon2){
const R = 6371;
let dLat = (lat2-lat1) * Math.PI/180;
let dLon = (lon2-lon1) * Math.PI/180;

let a = Math.sin(dLat/2)**2 +
Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180) *
Math.sin(dLon/2)**2;

return 2 * R * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}

// 🔥 MAIN FUNCTION (FIXED)
async function save(type){

const name=document.getElementById("name").value;
const file=document.getElementById("photo").files[0];

if(!name){
alert("Enter name");
return;
}

// 🔥 CHECK GPS
if(!navigator.geolocation){
alert("GPS not supported");
return;
}

navigator.geolocation.getCurrentPosition(async pos=>{

try{

let lat=pos.coords.latitude;
let lon=pos.coords.longitude;

let photoURL="";

// 🔥 SAFE PHOTO UPLOAD
if(file){
try{
let ref = storage.ref("photos/"+Date.now()+".jpg");
await ref.put(file);
photoURL = await ref.getDownloadURL();
}catch(e){
console.log("Photo upload failed", e);
}
}

// 🔥 GET LAST DATA
let km=0;
try{
let last = await db.collection("attendance")
.where("name","==",name)
.orderBy("timestamp","desc")
.limit(1).get();

if(!last.empty){
let prev = last.docs[0].data();
km = distance(prev.lat, prev.lon, lat, lon);
}
}catch(e){
console.log("KM calc error", e);
}

// 🔥 SAVE DATA
const now = new Date();

await db.collection("attendance").add({
name,
type,
lat,
lon,
photo:photoURL,
km,
timestamp: now.getTime(),
date: now.toLocaleDateString(),
time: now.toLocaleTimeString()
});

// 🔥 MAP
L.marker([lat,lon]).addTo(map)
.bindPopup(`${name}<br>${type}<br>${now.toLocaleTimeString()}`);

map.setView([lat,lon],17);

alert("✅ Saved successfully");

}catch(err){
console.error(err);
alert("❌ Error saving data");
}

}, err=>{
alert("❌ GPS error: enable location");
});

}

</script>

</body>
</html>
