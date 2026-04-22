<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field App</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{margin:0;font-family:Arial;}
.top{display:flex;padding:10px;gap:5px;background:#1e293b;}
input{flex:1;padding:10px;}
button{padding:10px;color:white;border:none;}
.in{background:green;}
.out{background:red;}
#map{height:80vh;}
#log{padding:10px;font-size:12px;background:#111;color:#0f0;height:20vh;overflow:auto;}
</style>
</head>

<body>

<div class="top">
<input id="name" placeholder="Name">
<button class="in" onclick="save('IN')">IN</button>
<button class="out" onclick="save('OUT')">OUT</button>
</div>

<div id="map"></div>
<div id="log">DEBUG LOG:</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

const firebaseConfig = {
  apiKey: "PASTE",
  authDomain: "PASTE",
  projectId: "PASTE"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// DEBUG
function log(msg){
document.getElementById("log").innerHTML += "<br>"+msg;
console.log(msg);
}

function gps(cb){
navigator.geolocation.getCurrentPosition(
p=>cb(p.coords.latitude,p.coords.longitude),
e=>log("❌ GPS ERROR: "+e.message)
);
}

function save(type){

const name=document.getElementById("name").value;
if(!name){log("❌ NO NAME");return;}

gps((lat,lon)=>{

const data={
name,type,
time:new Date().toLocaleString(),
lat,lon
};

log("📤 Sending to Firebase...");

db.collection("attendance").add(data)
.then(()=>{
log("✅ SAVED OK");
})
.catch(e=>{
log("❌ FIREBASE ERROR: "+e.message);
});

});

}

</script>

</body>
</html>
