<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>MDP Command Centre - Advanced</title>

<style>
body {font-family:Arial; margin:0; display:flex;}
.sidebar {width:240px; background:#f4f4f4; padding:15px;}
.nav-item {padding:10px; cursor:pointer;}
.nav-item:hover {background:#ddd;}
.main {flex:1; padding:20px;}
.view{display:none;}
.view.active{display:block;}
.card{border:1px solid #ddd; padding:15px; margin-bottom:15px; border-radius:8px;}
input, select{padding:6px; margin:4px;}
button{padding:6px 10px; cursor:pointer;}
.badge{padding:3px 8px; border-radius:10px; font-size:12px;}
.green{background:#d4f5e3;}
.amber{background:#ffe9c7;}
.red{background:#ffd6d6;}
</style>
</head>

<body>

<div class="sidebar">
  <h3>MDP Centre</h3>
  <div class="nav-item" onclick="show('dashboard')">Dashboard</div>
  <div class="nav-item" onclick="show('clients')">Clients</div>
  <div class="nav-item" onclick="show('revenue')">Revenue</div>
</div>

<div class="main">

<!-- DASHBOARD -->
<div id="dashboard" class="view active">
  <h2>Dashboard</h2>

  <div class="card">
    <h3>Next Best Actions</h3>
    <div id="actions"></div>
  </div>

  <div class="card">
    <h3>Forecast</h3>
    <div id="forecast"></div>
  </div>
</div>

<!-- CLIENTS -->
<div id="clients" class="view">
  <h2>Clients</h2>

  <div class="card">
    <h3>Add Client</h3>
    <input id="clientName" placeholder="Client Name">
    <input id="attendance" placeholder="Attendance %" type="number">
    <input id="nps" placeholder="NPS" type="number">
    <input id="progress" placeholder="Progress %" type="number">
    <input id="payment" placeholder="Payment Score" type="number">
    <button onclick="addClient()">Add</button>
  </div>

  <div class="card">
    <h3>Client List</h3>
    <div id="clientList"></div>
  </div>
</div>

<!-- REVENUE -->
<div id="revenue" class="view">
  <h2>Revenue Pipeline</h2>

  <div class="card">
    <h3>Add Deal</h3>
    <input id="dealClient" placeholder="Client">
    <input id="dealValue" placeholder="Value">
    <select id="dealStage">
      <option>Committed</option>
      <option>Likely</option>
      <option>At Risk</option>
    </select>
    <button onclick="addDeal()">Add Deal</button>
  </div>

  <div class="card">
    <h3>Deals</h3>
    <div id="dealList"></div>
  </div>
</div>

</div>

<script>

/* ---------------- NAV ---------------- */
function show(id){
  document.querySelectorAll(".view").forEach(v=>v.classList.remove("active"));
  document.getElementById(id).classList.add("active");
}

/* ---------------- STORAGE ---------------- */
let clients = JSON.parse(localStorage.getItem("clients")) || [];
let deals = JSON.parse(localStorage.getItem("deals")) || [];

/* ---------------- CLIENTS ---------------- */

function calcHealth(c){
  return Math.round(
    c.attendance*0.3 +
    c.nps*0.3 +
    c.progress*0.2 +
    c.payment*0.2
  );
}

function getHealthColor(score){
  if(score>75) return "green";
  if(score>50) return "amber";
  return "red";
}

function addClient(){
  let c = {
    name:clientName.value,
    attendance:+attendance.value,
    nps:+nps.value,
    progress:+progress.value,
    payment:+payment.value
  };
  clients.push(c);
  localStorage.setItem("clients",JSON.stringify(clients));
  renderClients();
}

function renderClients(){
  let html="";
  clients.forEach(c=>{
    let score = calcHealth(c);
    html += `
    <div>
      ${c.name} 
      <span class="badge ${getHealthColor(score)}">${score}</span>
    </div>`;
  });
  document.getElementById("clientList").innerHTML = html;
}

renderClients();

/* ---------------- DEALS ---------------- */

function addDeal(){
  let d = {
    client:dealClient.value,
    value:+dealValue.value,
    stage:dealStage.value
  };
  deals.push(d);
  localStorage.setItem("deals",JSON.stringify(deals));
  renderDeals();
  calcForecast();
}

function renderDeals(){
  let html="";
  deals.forEach(d=>{
    html += `<div>${d.client} - ₹${d.value} (${d.stage})</div>`;
  });
  document.getElementById("dealList").innerHTML = html;
}

renderDeals();

/* ---------------- FORECAST ---------------- */

function calcForecast(){
  let total=0;

  deals.forEach(d=>{
    if(d.stage==="Committed") total+=d.value;
    if(d.stage==="Likely") total+=d.value*0.6;
  });

  document.getElementById("forecast").innerText =
    "Forecast Revenue: ₹" + total;
}

calcForecast();

/* ---------------- ACTIONS ---------------- */

function renderActions(){
  let actions=[];

  clients.forEach(c=>{
    if(calcHealth(c)<50){
      actions.push("⚠ " + c.name + " is at risk");
    }
  });

  deals.forEach(d=>{
    if(d.stage==="At Risk"){
      actions.push("⚠ Deal at risk: " + d.client);
    }
  });

  if(actions.length===0){
    actions.push("✅ All good. Focus on growth.");
  }

  document.getElementById("actions").innerHTML =
    actions.map(a=>"<div>"+a+"</div>").join("");
}

renderActions();

</script>

</body>
</html>
