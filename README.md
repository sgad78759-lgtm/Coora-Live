<!DOCTYPE html><html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>منصة مباريات كرة القدم</title>
<style>
body{font-family:Arial;background:#020617;color:white;margin:0}
header{background:#0f172a;padding:15px;text-align:center}
h1{margin:0}
#date{font-size:14px;opacity:.8}.controls{display:flex;justify-content:center;gap:10px;padding:15px;flex-wrap:wrap} button,input,select{padding:10px;border-radius:8px;border:none;font-size:14px} button{background:#2563eb;color:white;cursor:pointer}

.container{max-width:1100px;margin:auto;padding:10px}

.card{background:#0f172a;margin:12px 0;padding:15px;border-radius:14px} .league{font-size:13px;opacity:.8;margin-bottom:8px}

.match{display:flex;align-items:center;justify-content:space-between;gap:10px}

.team{display:flex;align-items:center;gap:8px;font-weight:bold} .team img{width:28px;height:28px}

.score{font-size:18px;font-weight:bold}

.time{font-size:14px} .live{color:#22c55e;font-weight:bold}

.info{font-size:14px;margin-top:6px}

.stats{background:#020617;margin-top:10px;padding:10px;border-radius:8px;font-size:13px}

.section{background:#0f172a;margin-top:20px;border-radius:14px;padding:15px}

.table{width:100%;border-collapse:collapse} .table th,.table td{padding:6px;text-align:center;font-size:13px} .table th{opacity:.8}

footer{text-align:center;padding:20px;font-size:12px;opacity:.7} </style>

</head>
<body><header>
<h1>⚽ منصة مباريات كرة القدم</h1>
<p id="date"></p>
</header><div class="controls">
<button onclick="loadMatches()">🔄 تحديث</button>
<input id="search" placeholder="ابحث عن فريق" oninput="filterMatches()">
<select id="leagueFilter" onchange="filterMatches()">
<option value="all">كل الدوريات</option>
</select>
</div><div class="container"><div id="matches"></div><div class="section">
<h3>⭐ أفضل الهدافين (مثال)</h3>
<table class="table">
<tr>
<th>#</th>
<th>اللاعب</th>
<th>الفريق</th>
<th>الأهداف</th>
</tr>
<tr>
<td>1</td>
<td>Player A</td>
<td>Team A</td>
<td>22</td>
</tr>
<tr>
<td>2</td>
<td>Player B</td>
<td>Team B</td>
<td>19</td>
</tr>
<tr>
<td>3</td>
<td>Player C</td>
<td>Team C</td>
<td>17</td>
</tr>
</table>
</div><div class="section">
<h3>📋 ترتيب دوري (مثال)</h3>
<table class="table">
<tr>
<th>#</th>
<th>الفريق</th>
<th>لعب</th>
<th>نقاط</th>
</tr>
<tr>
<td>1</td>
<td>Team A</td>
<td>30</td>
<td>70</td>
</tr>
<tr>
<td>2</td>
<td>Team B</td>
<td>30</td>
<td>65</td>
</tr>
<tr>
<td>3</td>
<td>Team C</td>
<td>30</td>
<td>60</td>
</tr>
</table>
</div></div><footer>
الموقع يحدث البيانات تلقائياً كل 60 ثانية
</footer><script>
const matchesContainer=document.getElementById("matches");
const dateElement=document.getElementById("date");
const leagueFilter=document.getElementById("leagueFilter");

let allMatches=[];

function setDate(){
const now=new Date();
dateElement.innerText="📅 "+now.toLocaleDateString()+" - "+now.toLocaleTimeString();
}

async function loadMatches(){
try{
const today=new Date().toISOString().split("T")[0];

const res=await fetch(`https://www.thesportsdb.com/api/v1/json/3/eventsday.php?d=${today}&s=Soccer`);
const data=await res.json();

if(!data.events){
matchesContainer.innerHTML="لا توجد مباريات اليوم";
return;
}

allMatches=data.events.slice(0,40);
fillLeagueFilter();
renderMatches(allMatches);

}catch(e){
matchesContainer.innerHTML="حدث خطأ أثناء تحميل البيانات";
}
}

function fillLeagueFilter(){
const leagues=[...new Set(allMatches.map(m=>m.strLeague))];

leagueFilter.innerHTML='<option value="all">كل الدوريات</option>';

leagues.forEach(l=>{
const op=document.createElement("option");
op.value=l;
op.textContent=l;
leagueFilter.appendChild(op);
});
}

function renderMatches(matches){
matchesContainer.innerHTML="";

matches.forEach(match=>{

const home=match.strHomeTeam;
const away=match.strAwayTeam;
const league=match.strLeague;
const time=match.strTime || "--:--";
const status=match.strStatus || "";

const homeScore=match.intHomeScore ?? "-";
const awayScore=match.intAwayScore ?? "-";

const card=document.createElement("div");
card.className="card";

card.innerHTML=`
<div class="league">🏆 ${league}</div>

<div class="match">
<div class="team">⚽ ${home}</div>

<div class="score">
${homeScore} - ${awayScore}
</div>

<div class="team">${away} ⚽</div>
</div>

<div class="time">
${status==='Live' ? '<span class="live">🔴 مباشر</span>' : '🕒 '+time}
</div>

<div class="info">🎙️ المعلق: غير متوفر حالياً</div>
<div class="info">📺 القناة الناقلة: غير متوفر حالياً</div>

<div class="stats">
📊 إحصائيات سريعة<br>
استحواذ متوقع: 50% - 50%<br>
تسديدات متوقعة: 8 - 8
</div>
`;

matchesContainer.appendChild(card);

});
}

function filterMatches(){
const text=document.getElementById("search").value.toLowerCase();
const league=leagueFilter.value;

let filtered=allMatches.filter(m=>
(m.strHomeTeam+" "+m.strAwayTeam).toLowerCase().includes(text)
);

if(league!=="all"){
filtered=filtered.filter(m=>m.strLeague===league);
}

renderMatches(filtered);
}

setInterval(setDate,1000);
setInterval(loadMatches,60000);

setDate();
loadMatches();
</script></body>
</html>
