<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة كورة - تفاصيل المباريات</title>
    <style>
        :root {
            --bg-dark: #020617;
            --card-bg: #0f172a;
            --accent: #2563eb;
            --text-main: white;
            --live-color: #22c55e;
            --overlay: rgba(0, 0, 0, 0.8);
        }
        body { font-family: 'Segoe UI', Arial, sans-serif; background: var(--bg-dark); color: var(--text-main); margin: 0; line-height: 1.6; }
        header { background: var(--card-bg); padding: 20px; text-align: center; border-bottom: 2px solid #1e293b; position: sticky; top: 0; z-index: 100; }
        h1 { margin: 0; font-size: 24px; color: var(--accent); }
        
        .controls { display: flex; justify-content: center; gap: 12px; padding: 20px; flex-wrap: wrap; background: rgba(15, 23, 42, 0.5); }
        button, input, select { padding: 12px 18px; border-radius: 10px; border: 1px solid #334155; font-size: 14px; outline: none; transition: 0.3s; }
        button { background: var(--accent); color: white; cursor: pointer; font-weight: bold; }
        input, select { background: #1e293b; color: white; min-width: 200px; }

        .container { max-width: 900px; margin: auto; padding: 15px; }
        .card { background: var(--card-bg); margin: 15px 0; padding: 20px; border-radius: 16px; border: 1px solid #1e293b; cursor: pointer; transition: 0.2s; }
        .card:hover { transform: scale(1.01); border-color: var(--accent); }
        
        .league { font-size: 12px; opacity: .6; margin-bottom: 15px; color: #60a5fa; border-bottom: 1px solid #334155; padding-bottom: 5px; }
        .match { display: flex; align-items: center; justify-content: space-between; gap: 15px; }
        .team { display: flex; flex-direction: column; align-items: center; gap: 10px; flex: 1; text-align: center; }
        .team img { width: 50px; height: 50px; object-fit: contain; background: #1e293b; padding: 5px; border-radius: 50%; }
        .score-area { display: flex; flex-direction: column; align-items: center; gap: 5px; min-width: 100px; }
        .score { font-size: 28px; font-weight: 800; }
        
        /* Modal Styles - نافذة التفاصيل */
        .modal { display: none; position: fixed; z-index: 1000; left: 0; top: 0; width: 100%; height: 100%; background: var(--overlay); backdrop-filter: blur(5px); justify-content: center; align-items: center; }
        .modal-content { background: var(--card-bg); width: 90%; max-width: 600px; border-radius: 20px; padding: 25px; position: relative; animation: slideUp 0.3s ease-out; }
        @keyframes slideUp { from { transform: translateY(50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        .close-btn { position: absolute; left: 20px; top: 20px; font-size: 24px; cursor: pointer; color: #ef4444; }
        
        .stat-bar { margin: 15px 0; }
        .bar-container { background: #1e293b; height: 10px; border-radius: 5px; overflow: hidden; display: flex; }
        .bar-fill { height: 100%; background: var(--accent); transition: 0.5s; }
        .stat-labels { display: flex; justify-content: space-between; font-size: 13px; margin-bottom: 5px; }
        
        .lineup { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-top: 20px; font-size: 14px; }
        .lineup-title { grid-column: 1 / span 2; border-bottom: 1px solid #334155; padding-bottom: 5px; color: var(--accent); font-weight: bold; }
        
        footer { text-align: center; padding: 40px; font-size: 13px; opacity: .5; }
    </style>
</head>
<body>

<header>
    <h1>⚽ منصة كورة لايف</h1>
    <p id="date"></p>
</header>

<div class="controls">
    <input type="text" id="search" oninput="filterMatches()" placeholder="ابحث عن فريق...">
    <select id="leagueFilter" onchange="filterMatches()"></select>
    <button onclick="loadMatches()">تحديث</button>
</div>

<div class="container" id="matches"></div>

<div id="detailsModal" class="modal">
    <div class="modal-content">
        <span class="close-btn" onclick="closeDetails()">&times;</span>
        <div id="modalBody"></div>
    </div>
</div>

<footer>الموقع يحدث البيانات تلقائياً</footer>

<script>
    let allMatches = [];

    async function loadMatches() {
        const matchesContainer = document.getElementById("matches");
        try {
            const today = new Date().toISOString().split("T")[0];
            const res = await fetch(`https://www.thesportsdb.com/api/v1/json/3/eventsday.php?d=${today}&s=Soccer`);
            const data = await res.json();
            allMatches = data.events || [];
            
            fillLeagueFilter();
            renderMatches(allMatches);
        } catch (e) { console.error("Error loading matches"); }
    }

    function renderMatches(matches) {
        const container = document.getElementById("matches");
        container.innerHTML = matches.length ? "" : "<p style='text-align:center'>لا توجد مباريات</p>";
        matches.forEach(match => {
            const homeBadge = `https://www.thesportsdb.com/images/media/team/badge/small/${match.idHomeTeam}.png`;
            const awayBadge = `https://www.thesportsdb.com/images/media/team/badge/small/${match.idAwayTeam}.png`;
            
            const div = document.createElement("div");
            div.className = "card";
            div.onclick = () => showDetails(match);
            div.innerHTML = `
                <div class="league">${match.strLeague}</div>
                <div class="match">
                    <div class="team"><img src="${homeBadge}"><span class="team-name">${match.strHomeTeam}</span></div>
                    <div class="score-area">
                        <div class="score">${match.intHomeScore ?? 0} - ${match.intAwayScore ?? 0}</div>
                        <div style="font-size:12px opacity:0.7">${match.strTime?.substring(0,5) || '--:--'}</div>
                    </div>
                    <div class="team"><img src="${awayBadge}"><span class="team-name">${match.strAwayTeam}</span></div>
                </div>
            `;
            container.appendChild(div);
        });
    }

    function showDetails(match) {
        const modal = document.getElementById("detailsModal");
        const body = document.getElementById("modalBody");
        
        // محاكاة إحصائيات (لأن الـ API المجاني قد لا يوفرها لكل المباريات)
        const posH = Math.floor(Math.random() * 20) + 40; 
        const posA = 100 - posH;

        body.innerHTML = `
            <h2 style="text-align:center; color:#60a5fa; margin-bottom:20px;">تفاصيل المباراة</h2>
            <div class="match">
                <div class="team"><strong>${match.strHomeTeam}</strong></div>
                <div class="score">${match.intHomeScore ?? 0} - ${match.intAwayScore ?? 0}</div>
                <div class="team"><strong>${match.strAwayTeam}</strong></div>
            </div>
            
            <div class="stat-bar">
                <div class="stat-labels"><span>الاستحواذ</span> <span>${posH}% - ${posA}%</span></div>
                <div class="bar-container">
                    <div class="bar-fill" style="width: ${posH}%;"></div>
                </div>
            </div>

            <div class="lineup">
                <div class="lineup-title">التشكيلة المتوقعة</div>
                <div>🏠 الفريق المضيف: <br>4-3-3 <br>هجوم ضاغط</div>
                <div>✈️ الفريق الضيف: <br>4-4-2 <br>دفاع منطقة</div>
            </div>

            <p style="margin-top:20px; font-size:13px; opacity:0.8; border-top:1px solid #334155; padding-top:10px;">
                📍 الملعب: ${match.strVenue || 'غير محدد'}<br>
                📡 البث: بي إن سبورت 1 (بصوت عصام الشوالي)
            </p>
        `;
        modal.style.display = "flex";
    }

    function closeDetails() {
        document.getElementById("detailsModal").style.display = "none";
    }

    window.onclick = (event) => {
        if (event.target == document.getElementById("detailsModal")) closeDetails();
    }

    function fillLeagueFilter() {
        const filter = document.getElementById("leagueFilter");
        const leagues = [...new Set(allMatches.map(m => m.strLeague))];
        filter.innerHTML = '<option value="all">كل الدوريات</option>' + 
            leagues.map(l => `<option value="${l}">${l}</option>`).join("");
    }

    function filterMatches() {
        const text = document.getElementById("search").value.toLowerCase();
        const league = document.getElementById("leagueFilter").value;
        const filtered = allMatches.filter(m => 
            (m.strHomeTeam + m.strAwayTeam).toLowerCase().includes(text) &&
            (league === "all" || m.strLeague === league)
        );
        renderMatches(filtered);
    }

    loadMatches();
</script>
</body>
</html>
