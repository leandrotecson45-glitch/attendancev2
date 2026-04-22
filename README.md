<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Field Attendance</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body { margin:0; font-family: Arial; background:#0f172a; color:white; }

.topbar {
    display:flex;
    gap:5px;
    padding:10px;
    background:#1e293b;
}

input {
    flex:1;
    padding:10px;
    border-radius:8px;
    border:none;
}

button {
    padding:10px;
    border:none;
    border-radius:8px;
    color:white;
    font-weight:bold;
}

.timein { background:#22c55e; }
.timeout { background:#ef4444; }

#map { height:calc(100vh - 60px); }
</style>
</head>

<body>

<div class="topbar">
    <input type="text" id="name" placeholder="Enter Name">
    <button class="timein" onclick="timeIn()">IN</button>
    <button class="timeout" onclick="timeOut()">OUT</button>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>

<script>

// 🔥 PALITAN MO ITO NG SARILI MONG CONFIG
const firebaseConfig = {
  apiKey: "AIzaSyDZ2YOn7k1h5kSUppZcWfZ5gAvJlaOVVuA",
  authDomain: "attendance1-697b2.firebaseapp.com",
  projectId: "attendance1-697b2",
  storageBucket: "attendance1-697b2.firebasestorage.app",
  messagingSenderId: "911862803622",
  appId: "1:911862803622:web:63f0bafabaec74f2665867",
  measurementId: "G-WR38DZW80B"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// MAP
const map = L.map('map').setView([15.5,120.9],13);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// ADD MARKER
function addMarker(data){
    const marker = L.marker([data.lat, data.lon]).addTo(map);

    marker.bindPopup(`
        <b>${data.name}</b><br>
        ${data.type}<br>
        🕒 ${data.time}
    `);
}

// GPS
function getLocation(callback){
    navigator.geolocation.getCurrentPosition(
        pos => callback(pos.coords.latitude,pos.coords.longitude),
        err => alert("Enable GPS + allow permission"),
        { enableHighAccuracy:true }
    );
}

// SAVE
function save(type){
    const name = document.getElementById("name").value;
    if(!name) return alert("Enter name");

    getLocation((lat,lon)=>{

        const record = {
            name:name,
            type:type,
            time:new Date().toLocaleString(),
            lat:lat,
            lon:lon
        };

        // 🔥 SAVE TO FIREBASE
        db.collection("attendance").add(record);

        addMarker(record);

        map.setView([lat,lon],17);
    });
}

function timeIn(){ save("TIME IN"); }
function timeOut(){ save("TIME OUT"); }

</script>

</body>
</html>
