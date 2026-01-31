<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>aletheia website</title>
<style>
* { margin:0; padding:0; box-sizing:border-box; font-family:sans-serif; }
html, body {
  width:100%; height:100%;
  color:white; text-align:center;
  background:linear-gradient(135deg,#151576,#e62801,#f9ca22);
  overflow:hidden;
}
.screen {
  position:fixed; width:100vw; height:100vh;
  display:none; flex-direction:column;
  justify-content:center; align-items:center;
}
button {
  background:#e62801; color:white;
  border:none; padding:15px 35px;
  margin:8px; font-size:16px;
  border-radius:10px; cursor:pointer;
}
button:hover { background:#f9ca22; color:#151576; }
.section-box { background:rgba(0,0,0,.25); padding:15px; margin:10px; width:85%; border-radius:10px; }
.mission-item { background:rgba(255,255,255,.15); padding:12px; margin:10px; width:80%; border-radius:8px; text-align:left; }
input { padding:10px; border-radius:8px; border:none; margin:5px; }
</style>
</head>

<body>

<!-- VIDEO SCREEN -->
<div id="videoScreen" class="screen" style="display:flex">
  <video autoplay muted loop playsinline id="introVideo" style="width:100%;height:100%;object-fit:cover;">
    <source src="cinematography.mp4" type="video/mp4">
  </video>
</div>

<!-- HOME -->
<div id="home" class="screen">
  <h1>aletheia website</h1>
  <button onclick="loggedUser ? showScreen('mission') : showScreen('login')">Mission</button>
  <button onclick="showScreen('rewards')">Rewards</button>
  <button onclick="showScreen('disposalMenu')">comments</button>
</div>

<!-- LOGIN -->
<div id="login" class="screen">
  <h2>Mission Login</h2>
  <input id="username" placeholder="Enter your name">
  <button onclick="login()">Login</button>
  <button onclick="showScreen('home')">Back</button>
</div>

<!-- MISSION -->
<div id="mission" class="screen">
  <h2>Missions</h2>

  <div class="mission-item">
    Report Illegal Waste (5 pts)
    <input type="file" accept="image/*" onchange="completeMission(this,'illegal',5)">
  </div>

  <div class="mission-item">
    Scan EcoWatch QR (5 pts)
    <input type="file" accept="image/*" onchange="completeMission(this,'qr',5)">
  </div>

  <div class="mission-item">
    Proper Waste Disposal (10 pts)
    <input type="file" accept="image/*" onchange="completeMission(this,'proper',10)">
  </div>

  <button onclick="showScreen('home')">Home</button>
</div>

<!-- REWARDS -->
<div id="rewards" class="screen">
  <h1>Rewards</h1>
  <h2>Points: <span id="points">0</span></h2>

  <div class="section-box">Sticker – 10 pts <button onclick="redeem(10)">Redeem</button></div>
  <div class="section-box">Eco Bag – 20 pts <button onclick="redeem(20)">Redeem</button></div>
  <div class="section-box">Tumbler – 30 pts <button onclick="redeem(30)">Redeem</button></div>
  <button onclick="showScreen('home')">Home</button>
</div>

<!-- DISPOSAL MENU -->
<div id="disposalMenu" class="screen">
  <h2>Waste Disposal</h2>
  <div id="companyInputDiv">
    <input id="companyName" placeholder="Company Name">
    <button onclick="submitCompany()">Submit</button>
  </div>
  <div id="wasteButtons" style="display:none; flex-direction: column;">
    <button onclick="logWeight('Biodegradable')">Biodegradable</button>
    <button onclick="logWeight('Non-Biodegradable')">Non-Biodegradable</button>
    <button onclick="logWeight('Special Waste')">Special Waste</button>
    <button onclick="showScreen('recordsInput')">Book Record</button>
    <button onclick="showScreen('home')">Home</button>
  </div>
</div>

<!-- RECORDS INPUT -->
<div id="recordsInput" class="screen">
  <input id="recordCompanyName" placeholder="Company Name">
  <button onclick="showRecordsMenu()">Submit</button>
</div>

<div id="recordsMenu" class="screen">
  <h2 id="displayCompany"></h2>
  <button onclick="showRecords('daily')">Daily</button>
  <button onclick="showRecords('weekly')">Weekly</button>
  <button onclick="showRecords('monthly')">Monthly</button>
  <button onclick="showRecords('annually')">Annually</button>
</div>

<div id="records" class="screen">
  <h2 id="periodTitle"></h2>
  <div id="recordsDisplay" class="section-box"></div>
  <button onclick="showScreen('recordsMenu')">Back</button>
</div>

<script>
/* ---------- GLOBAL ---------- */
let users = JSON.parse(localStorage.getItem("users")) || {};
let loggedUser = localStorage.getItem("loggedUser") || null;
let totalPoints = 0;

let disposals = JSON.parse(localStorage.getItem("disposals")) || [];
let bookOfRecords = JSON.parse(localStorage.getItem("bookOfRecords")) || [];

/* ---------- SCREEN ---------- */
function showScreen(id){
  document.querySelectorAll('.screen').forEach(s=>s.style.display='none');
  document.getElementById(id).style.display='flex';
  resetTimer();
}

/* ---------- VIDEO CLICK ---------- */
const video = document.getElementById("introVideo");
video.addEventListener("click", () => showScreen("home"));
document.addEventListener("click", (e) => {
  const videoScreen = document.getElementById("videoScreen");
  if(videoScreen.style.display === "flex" && e.target !== document.getElementById("username"))
    showScreen("home");
});

/* ---------- INACTIVITY TIMER ---------- */
let inactivityTimer;
function resetTimer(){
  clearTimeout(inactivityTimer);
  inactivityTimer = setTimeout(()=>showScreen('videoScreen'),10000);
}
document.addEventListener("click", resetTimer);
document.addEventListener("mousemove", resetTimer);
document.addEventListener("keydown", resetTimer);

/* ---------- LOGIN ---------- */
function login(){
  const name = username.value.trim();
  if(!name) return alert("Enter your name");
  if(!users[name]) users[name] = { points:0, completed:[] };
  loggedUser = name;
  localStorage.setItem("users", JSON.stringify(users));
  localStorage.setItem("loggedUser", loggedUser);
  syncPoints();
  showScreen("mission");
}

/* ---------- MISSIONS ---------- */
function completeMission(input, id, pts){
  if(!loggedUser) return alert("Login first");
  if(users[loggedUser].completed.includes(id)) return alert("Already completed");
  users[loggedUser].completed.push(id);
  users[loggedUser].points += pts;
  localStorage.setItem("users", JSON.stringify(users));
  syncPoints();
  alert("Mission completed! +" + pts);
}

/* ---------- REWARDS ---------- */
function redeem(cost){
  if(!loggedUser) return alert("Login first");
  if(users[loggedUser].points < cost) return alert("Not enough points");
  users[loggedUser].points -= cost;
  localStorage.setItem("users", JSON.stringify(users));
  syncPoints();
  alert("Reward redeemed!");
}

/* ---------- SYNC POINTS ---------- */
function syncPoints(){
  totalPoints = loggedUser ? users[loggedUser].points : 0;
  document.getElementById("points").innerText = totalPoints;
}

/* ---------- DISPOSAL ---------- */
let currentCompany="", recordCompany="";

function submitCompany(){
  currentCompany = companyName.value.trim();
  if(!currentCompany) return alert("Enter company name");
  wasteButtons.style.display="flex";
}

function logWeight(type){
  let w = prompt("Enter kg:");
  if(!w || isNaN(w) || w<=0) return alert("Invalid weight");
  disposals.push({
    company: currentCompany,
    type, weight:w,
    date: new Date().toISOString().split("T")[0]
  });
  // Book of Record ONLY logs disposal
  bookOfRecords.push({
    company: currentCompany,
    type,
    weight: w,
    date: new Date().toLocaleString()
  });
  localStorage.setItem("disposals", JSON.stringify(disposals));
  localStorage.setItem("bookOfRecords", JSON.stringify(bookOfRecords));
  alert(`${type} disposal of ${w} kg recorded for ${currentCompany}`);
}

/* ---------- RECORDS ---------- */
function showRecordsMenu(){
  recordCompany = recordCompanyName.value.trim();
  if(!recordCompany) return alert("Enter company name");
  displayCompany.innerText = recordCompany;
  showScreen("recordsMenu");
}

function showRecords(period){
  const now = new Date();
  const companyDisposals = disposals.filter(d => d.company === recordCompany);
  document.getElementById("periodTitle").innerText = period.toUpperCase();
  document.getElementById("recordsDisplay").innerHTML =
    companyDisposals.map(d => `${d.date} – ${d.type} – ${d.weight}kg`).join("<br>") || "No records";
  showScreen("records");
}

/* ---------- SECURITY ---------- */
document.addEventListener('contextmenu', e=>e.preventDefault());
document.addEventListener('keydown', e=>{
  if(e.key==='F12'||(e.ctrlKey && ['t','n','w','r'].includes(e.key.toLowerCase()))) e.preventDefault();
});
</script>

</body>
</html>
