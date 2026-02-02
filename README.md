# coaches
<!DOCTYPE html>
<html lang="en" dir="ltr" id="mainHtml">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tactical AI | Professional Bilingual Dashboard</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #0f172a;
            --card: #1e293b;
            --team-l: #38bdf8;
            --team-r: #ef4444;
            --ai-scan: #22c55e;
            --text: #f8fafc;
            --dim: #94a3b8;
        }

        [dir="rtl"] { font-family: 'Cairo', sans-serif; }
        [dir="ltr"] { font-family: 'Segoe UI', sans-serif; }

        body { background-color: var(--bg); color: var(--text); margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; }
        .dashboard-container { width: 100%; max-width: 1400px; }

        /* HEADER & UI */
        .header-ui { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .lang-switcher { display: flex; gap: 5px; background: var(--card); padding: 5px; border-radius: 8px; border: 1px solid #334155; }
        .lang-btn { padding: 5px 15px; border-radius: 5px; border: none; cursor: pointer; background: transparent; color: var(--dim); font-weight: bold; }
        .lang-btn.active { background: var(--team-l); color: #000; }

        .possession-container {
            width: 100%; background: #1e293b; height: 35px; border-radius: 8px; 
            margin-bottom: 20px; overflow: hidden; display: flex; border: 1px solid #334155;
        }
        .possession-bar { height: 100%; transition: width 1s ease; display: flex; align-items: center; padding: 0 15px; font-weight: bold; font-size: 13px; width: 50%; }
        .possession-l { background: var(--team-l); color: #000; }
        .possession-r { background: var(--team-r); color: #fff; justify-content: flex-end; flex-grow: 1; }

        /* VIDEO */
        .viewport {
            position: relative; width: 100%; aspect-ratio: 16 / 9;
            background: #000; border-radius: 12px; overflow: hidden; border: 2px solid #334155;
        }
        #videoPlayer { width: 100%; height: 100%; object-fit: cover; }
        .tracker { position: absolute; width: 80px; height: 80px; border-radius: 50%; border: 3px solid var(--ai-scan); pointer-events: none; display: none; transform: translate(-50%, -50%); z-index: 10; }

        /* ANALYSIS GRID */
        .analysis-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; margin-top: 25px; }
        .team-card { background: var(--card); border-radius: 12px; padding: 20px; border-top: 6px solid #334155; opacity: 0.5; transition: all 0.5s; }
        .team-card.active { opacity: 1; border-top-width: 10px; }

        /* HD IDENTITY CAM */
        .zoom-cam-container { position: relative; width: 100%; height: 350px; background: #000; border-radius: 10px; overflow: hidden; border: 2px solid #444; }
        canvas.zoom-canvas { width: 100%; height: 100%; }
        
        /* CALIBRATION CONTROLS */
        .calibration { background: rgba(0,0,0,0.4); padding: 15px; border-radius: 8px; margin: 15px 0; display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .calibration label { font-size: 10px; color: var(--dim); display: block; margin-bottom: 5px; text-transform: uppercase; }
        input[type=range] { width: 100%; accent-color: var(--ai-scan); }

        /* INSIGHT ITEMS */
        .insight-item { background: rgba(15, 23, 42, 0.5); padding: 12px; border-radius: 6px; margin-bottom: 10px; border-left: 4px solid #334155; }
        [dir="rtl"] .insight-item { border-left: none; border-right: 4px solid #334155; }
        .insight-item h4 { margin: 0 0 5px 0; font-size: 11px; text-transform: uppercase; color: var(--dim); }
        .insight-item p { margin: 0; font-size: 13px; line-height: 1.4; color: var(--text); }

        /* METRICS */
        .metric-row { display: flex; justify-content: space-between; font-size: 11px; margin-bottom: 4px; }
        .progress-bg { background: #0f213a; height: 8px; border-radius: 4px; overflow: hidden; margin-bottom: 15px; }
        .progress-fill { height: 100%; width: 0%; transition: width 1.5s ease; }
        
        .visual-row { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 15px; }
        .mini-screen { background: #0f172a; border-radius: 8px; height: 110px; border: 1px solid #334155; position: relative; overflow: hidden; }
        
        .btn { padding: 12px 24px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; text-transform: uppercase; font-family: inherit; }
    </style>
</head>
<body>

<div class="dashboard-container">
    <header class="header-ui">
        <div>
            <h1 style="margin:0;" id="txt-title">Tactical AI Summary</h1>
            <p style="color:var(--dim); margin:0;" id="txt-subtitle">Full Automated Player ID & Strategic Report</p>
        </div>
        <div style="display:flex; gap:12px; align-items: center;">
            <div class="lang-switcher">
                <button id="btn-en" class="lang-btn active" onclick="setLanguage('en')">EN</button>
                <button id="btn-ar" class="lang-btn" onclick="setLanguage('ar')">AR</button>
            </div>
            <input type="file" id="videoUpload" accept="video/*" style="display:none;">
            <button class="btn" style="background:var(--team-l); color:#000;" onclick="document.getElementById('videoUpload').click()" id="txt-upload">Upload Video</button>
            <button class="btn" style="background:var(--ai-scan); color:#000;" onclick="startAnalysis()" id="txt-analyze">Start Analysis</button>
        </div>
    </header>

    <div class="possession-container">
        <div class="possession-bar possession-l" id="possL">TEAM LEFT: 50%</div>
        <div class="possession-bar possession-r" id="possR">50% :TEAM RIGHT</div>
    </div>

    <div class="viewport">
        <video id="videoPlayer" muted></video>
        <div id="trackerL" class="tracker"></div>
        <div id="trackerR" class="tracker"></div>
    </div>

    <div class="analysis-grid">
        <!-- TEAM LEFT -->
        <div class="team-card" id="cardL">
            <h3 style="color:var(--team-l); margin-top:0;" id="txt-cardL-title">TEAM LEFT (BLUE) | TARGET ID</h3>
            <div class="zoom-cam-container"><canvas id="canvasL" class="zoom-canvas"></canvas></div>
            
            <div class="calibration">
                <div><label id="txt-panXL">Center Target (X)</label><input type="range" id="panXL" min="-100" max="100" value="0" oninput="drawLens('L')"></div>
                <div><label id="txt-panYL">Center Target (Y)</label><input type="range" id="panYL" min="-100" max="100" value="0" oninput="drawLens('L')"></div>
                <div style="grid-column: span 2;"><label id="txt-zoomL">Lens Magnification</label><input type="range" id="zoomL" min="50" max="500" value="180" oninput="drawLens('L')"></div>
            </div>

            <div class="insight-section">
                <div class="insight-item" id="itemL1">
                    <h4 id="txt-targetL">Target Analysis (Player #10)</h4>
                    <p id="val-targetL">Scanning...</p>
                </div>
                <div class="insight-item" id="itemL2">
                    <h4 id="txt-strengthL">Tactical Strength</h4>
                    <p id="val-strengthL">Scanning...</p>
                </div>
                <div class="insight-item" id="itemL3">
                    <h4 id="txt-focusL">Focus Area</h4>
                    <p id="val-focusL">Scanning...</p>
                </div>
            </div>
            
            <div class="metric-row"><span id="txt-pressL">Pressing Efficiency</span><span id="pValL">0%</span></div>
            <div class="progress-bg"><div class="progress-fill" id="pBarL" style="background: var(--team-l);"></div></div>

            <div class="visual-row">
                <div class="mini-screen" id="heatL"></div>
                <div class="mini-screen"><canvas id="netL" style="width:100%; height:100%"></canvas></div>
            </div>
        </div>

        <!-- TEAM RIGHT -->
        <div class="team-card" id="cardR">
            <h3 style="color:var(--team-r); margin-top:0;" id="txt-cardR-title">TEAM RIGHT (RED) | TARGET ID</h3>
            <div class="zoom-cam-container"><canvas id="canvasR" class="zoom-canvas"></canvas></div>

            <div class="calibration">
                <div><label id="txt-panXR">Center Target (X)</label><input type="range" id="panXR" min="-100" max="100" value="0" oninput="drawLens('R')"></div>
                <div><label id="txt-panYR">Center Target (Y)</label><input type="range" id="panYR" min="-100" max="100" value="0" oninput="drawLens('R')"></div>
                <div style="grid-column: span 2;"><label id="txt-zoomR">Lens Magnification</label><input type="range" id="zoomR" min="50" max="500" value="180" oninput="drawLens('R')"></div>
            </div>

            <div class="insight-section">
                <div class="insight-item" id="itemR1">
                    <h4 id="txt-targetR">Target Analysis (Player #4)</h4>
                    <p id="val-targetR">Scanning...</p>
                </div>
                <div class="insight-item" id="itemR2">
                    <h4 id="txt-strengthR">Tactical Strength</h4>
                    <p id="val-strengthR">Scanning...</p>
                </div>
                <div class="insight-item" id="itemR3">
                    <h4 id="txt-focusR">Focus Area</h4>
                    <p id="val-focusR">Scanning...</p>
                </div>
            </div>

            <div class="metric-row"><span id="txt-pressR">Pressing Efficiency</span><span id="pValR">0%</span></div>
            <div class="progress-bg"><div class="progress-fill" id="pBarR" style="background: var(--team-r);"></div></div>

            <div class="visual-row">
                <div class="mini-screen" id="heatR"></div>
                <div class="mini-screen"><canvas id="netR" style="width:100%; height:100%"></canvas></div>
            </div>
        </div>
    </div>
</div>

<script>
    const translations = {
        en: {
            title: "Tactical AI Summary",
            subtitle: "Full Automated Player ID & Strategic Report",
            upload: "Upload Video",
            analyze: "Start Analysis",
            cardL: "TEAM LEFT (BLUE) | TARGET ID",
            cardR: "TEAM RIGHT (RED) | TARGET ID",
            pan: "Center Target",
            zoom: "Lens Magnification",
            target: "Target Analysis",
            strength: "Tactical Strength",
            focus: "Focus Area",
            pressing: "Pressing Efficiency",
            possL: "TEAM LEFT",
            possR: "TEAM RIGHT",
            l_target: "Player #10 identified as High-Value Target. Orientation: Attacking half-space. Passing Success: 92%.",
            l_strength: "Elite spatial exploitation. Consistently creates 3v2 overloads on the left channel.",
            l_focus: "Defensive tracking. Player is leaving a 12m gap during the transition from attack to defense.",
            r_target: "Player #4 identified as Defensive Anchor. Orientation: Deep Block. Duel Win Rate: 78%.",
            r_strength: "Superior vertical compactness. Maintaining effective offside trap depth at 32m.",
            r_focus: "Midfield support. Defender is being forced into 1v2 situations due to pivot isolation."
        },
        ar: {
            title: "ملخص تكتيكي بالذكاء الاصطناعي",
            subtitle: "تحديد تلقائي للاعبين وتقارير استراتيجية كاملة",
            upload: "تحميل الفيديو",
            analyze: "بدء التحليل",
            cardL: "الفريق الأيسر (الأزرق) | تحديد الهدف",
            cardR: "الفريق الأيمن (الأحمر) | تحديد الهدف",
            pan: "توسيط الهدف",
            zoom: "تكبير العدسة",
            target: "تحليل الهدف",
            strength: "القوة التكتيكية",
            focus: "منطقة التركيز",
            pressing: "كفاءة الضغط",
            possL: "الفريق الأيسر",
            possR: "الفريق الأيمن",
            l_target: "تم تحديد اللاعب رقم 10 كهدف عالي القيمة. التوجه: نصف مساحة هجومية. نجاح التمرير: 92%.",
            l_strength: "استغلال مساحات متميز. يخلق باستمرار تفوقاً عددياً 3 ضد 2 في الرواق الأيسر.",
            l_focus: "التغطية الدفاعية. يترك اللاعب فجوة 12 متراً أثناء التحول من الهجوم إلى الدفاع.",
            r_target: "تم تحديد اللاعب رقم 4 كركيزة دفاعية. التوجه: دفاع متأخر. معدل الفوز بالمواجهات: 78%.",
            r_strength: "تماسك رأسي متفوق. الحفاظ على عمق فعال لمصيدة التسلل عند 32 متراً.",
            r_focus: "دعم خط الوسط. المدافع يضطر لمواقف 1 ضد 2 بسبب عزل لاعب الارتكاز."
        }
    };

    let currentLang = 'en';
    const video = document.getElementById('videoPlayer');
    const teams = {
        L: { canvas: document.getElementById('canvasL'), ctx: document.getElementById('canvasL').getContext('2d'), tracker: document.getElementById('trackerL'), found: false, x: 30, y: 40, color: '#38bdf8' },
        R: { canvas: document.getElementById('canvasR'), ctx: document.getElementById('canvasR').getContext('2d'), tracker: document.getElementById('trackerR'), found: false, x: 70, y: 55, color: '#ef4444' }
    };

    function setLanguage(lang) {
        currentLang = lang;
        document.getElementById('mainHtml').dir = lang === 'ar' ? 'rtl' : 'ltr';
        document.getElementById('btn-en').classList.toggle('active', lang === 'en');
        document.getElementById('btn-ar').classList.toggle('active', lang === 'ar');

        // Static UI
        const t = translations[lang];
        document.getElementById('txt-title').innerText = t.title;
        document.getElementById('txt-subtitle').innerText = t.subtitle;
        document.getElementById('txt-upload').innerText = t.upload;
        document.getElementById('txt-analyze').innerText = t.analyze;
        document.getElementById('txt-cardL-title').innerText = t.cardL;
        document.getElementById('txt-cardR-title').innerText = t.cardR;
        
        document.getElementById('txt-panXL').innerText = t.pan + " (X)";
        document.getElementById('txt-panYL').innerText = t.pan + " (Y)";
        document.getElementById('txt-zoomL').innerText = t.zoom;
        document.getElementById('txt-panXR').innerText = t.pan + " (X)";
        document.getElementById('txt-panYR').innerText = t.pan + " (Y)";
        document.getElementById('txt-zoomR').innerText = t.zoom;

        document.getElementById('txt-targetL').innerText = t.target + " (#10)";
        document.getElementById('txt-strengthL').innerText = t.strength;
        document.getElementById('txt-focusL').innerText = t.focus;
        document.getElementById('txt-pressL').innerText = t.pressing;

        document.getElementById('txt-targetR').innerText = t.target + " (#4)";
        document.getElementById('txt-strengthR').innerText = t.strength;
        document.getElementById('txt-focusR').innerText = t.focus;
        document.getElementById('txt-pressR').innerText = t.pressing;

        // Dynamic Data (if already found)
        if(teams.L.found) updateDynamicText('L');
        if(teams.R.found) updateDynamicText('R');
        updatePossessionUI();
    }

    function updateDynamicText(id) {
        const t = translations[currentLang];
        document.getElementById(`val-target${id}`).innerText = t[`${id.toLowerCase()}_target`];
        document.getElementById(`val-strength${id}`).innerText = t[`${id.toLowerCase()}_strength`];
        document.getElementById(`val-focus${id}`).innerText = t[`${id.toLowerCase()}_focus`];
    }

    function updatePossessionUI() {
        const t = translations[currentLang];
        const valL = teams.R.found ? 63 : 50;
        const valR = 100 - valL;
        document.getElementById('possL').style.width = valL + '%';
        document.getElementById('possL').innerText = `${t.possL}: ${valL}%`;
        document.getElementById('possR').innerText = `${valR}% :${t.possR}`;
    }

    // DISCOVERY LOGIC
    document.getElementById('videoUpload').addEventListener('change', function(e) {
        const file = e.target.files[0];
        if (file) video.src = URL.createObjectURL(file);
    });

    let isScanning = false;
    function startAnalysis() {
        if(!video.src) return;
        isScanning = true;
        teams.L.found = teams.R.found = false;
        video.currentTime = 0;
        video.play();
        teams.L.tracker.style.display = teams.R.tracker.style.display = 'block';
    }

    video.addEventListener('timeupdate', () => {
        if(!isScanning) return;
        const t = video.currentTime;
        if(t >= 4.0 && !teams.L.found) lock('L');
        else if(!teams.L.found) move('L', 25 + Math.sin(t*2)*10, 45 + Math.cos(t)*5);

        if(t >= 8.0 && !teams.R.found) lock('R');
        else if(!teams.R.found) move('R', 75 + Math.cos(t*1.5)*10, 50 + Math.sin(t*2)*5);
    });

    function move(id, x, y) {
        teams[id].x = x; teams[id].y = y;
        teams[id].tracker.style.left = x + '%';
        teams[id].tracker.style.top = y + '%';
        drawLens(id);
    }

    function lock(id) {
        video.pause();
        teams[id].found = true;
        const team = teams[id];
        team.tracker.style.borderColor = team.color;
        team.tracker.style.boxShadow = `0 0 30px ${team.color}`;
        document.getElementById(`card${id}`).classList.add('active');
        document.getElementById(`card${id}`).style.borderTopColor = team.color;
        
        updateDynamicText(id);
        drawLens(id);
        
        // Metrics & Visuals
        const pVal = id === 'L' ? 84 : 62;
        document.getElementById(`pBar${id}`).style.width = pVal + '%';
        document.getElementById(`pVal${id}`).innerText = pVal + '%';
        document.getElementById(`heat${id}`).style.background = `radial-gradient(circle at center, ${team.color}66, transparent 70%)`;
        drawNet(id, team.color);

        if(id === 'R') updatePossessionUI();
        setTimeout(() => { if(isScanning) video.play(); }, 3000);
    }

    function drawLens(id) {
        const team = teams[id];
        const canvas = team.canvas;
        const ctx = team.ctx;
        const pX = parseInt(document.getElementById(`panX${id}`).value);
        const pY = parseInt(document.getElementById(`panY${id}`).value);
        const zm = parseInt(document.getElementById(`zoom${id}`).value);

        canvas.width = 650; canvas.height = 650;
        const realX = ((team.x + (pX/6)) / 100) * video.videoWidth;
        const realY = ((team.y + (pY/6)) / 100) * video.videoHeight;
        const size = 650 / (zm / 100);

        ctx.fillStyle = "#000"; ctx.fillRect(0,0,650,650);
        ctx.drawImage(video, realX-(size/2), realY-(size/2), size, size, 0, 0, 650, 650);
        ctx.strokeStyle = team.found ? team.color : "#22c55e";
        ctx.lineWidth = 5; ctx.beginPath(); ctx.arc(325, 325, 100, 0, Math.PI*2); ctx.stroke();
    }

    function drawNet(id, color) {
        const canvas = document.getElementById(`net${id}`);
        const ctx = canvas.getContext('2d');
        canvas.width = 300; canvas.height = 110;
        const nodes = [];
        for(let i=0; i<6; i++) nodes.push({x: Math.random()*260+20, y: Math.random()*80+20});
        ctx.strokeStyle = color + '44';
        nodes.forEach((n, i) => { nodes.forEach((n2, j) => { if(i!==j && Math.random()>0.5){ctx.beginPath(); ctx.moveTo(n.x,n.y); ctx.lineTo(n2.x,n2.y); ctx.stroke();}});});
        nodes.forEach(n => { ctx.fillStyle = color; ctx.beginPath(); ctx.arc(n.x,n.y,4,0,Math.PI*2); ctx.fill();});
    }

    // Initialize English
    setLanguage('en');
</script>

</body>
</html>
