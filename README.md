<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;background:#0b1220;color:white;}
.header{padding:15px;background:#0f172a;}
input{width:100%;padding:14px;border-radius:10px;border:none;margin-bottom:10px;}
.buttons{display:flex;gap:10px;}
button{flex:1;padding:15px;border:none;border-radius:12px;color:white;font-weight:bold;}
.in{background:#22c55e;}
.out{background:#ef4444;}
#map{height:70vh;}
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

// 🔥 CONFIG
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

// SAVE FUNCTION
async function save(type){

const name = document.getElementById("name").value;
const file = document.getElementById("photo").files[0];

if(!name){
alert("Enter name");
return;
}

navigator.geolocation.getCurrentPosition(async pos=>{

let lat = pos.coords.latitude;
let lon = pos.coords.longitude;

let photoURL = "";

// 🔥 SAFE UPLOAD (hindi sisira kahit mag fail)
if(file){
try{
let ref = storage.ref("photos/"+Date.now()+".jpg");
await ref.put(file);
photoURL = await ref.getDownloadURL();
}catch(e){
console.log("Photo upload failed");
}
}

const now = new Date();

try{

await db.collection("attendance").add({
name,
type,
lat,
lon,
photo: photoURL,
time: now.toLocaleTimeString(),
date: now.toLocaleDateString(),
timestamp: now.getTime()
});

L.marker([lat,lon]).addTo(map)
.bindPopup(`
<b>${name}</b><br>
${type}<br>
${now.toLocaleTimeString()}
`);

map.setView([lat,lon],17);

alert("✅ SUCCESS");

}catch(e){
alert("❌ SAVE ERROR");
}

}, err=>{
alert("❌ GPS ERROR");
});

}

</script>

</body>
</html>
