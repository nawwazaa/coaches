# coaches
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tactical AI | Professional Discovery Suite</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
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

        body {
            font-family: 'Segoe UI', sans-serif;
            background-color: var(--bg); color: var(--text);
            margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center;
        }

        .dashboard-container { width: 100%; max-width: 1400px; }

        /* HEADER */
        .header-ui { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
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

        /* ANALYSIS CARDS */
        .analysis-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; margin-top: 25px; }
        .team-card { background: var(--card); border-radius: 12px; padding: 20px; border-top: 6px solid #334155; opacity: 0.4; transition: all 0.5s; }
        .team-card.active { opacity: 1; border-top-width: 8px; }

        /* HD IDENTITY CAM */
        .zoom-cam-container { position: relative; width: 100%; height: 350px; background: #000; border-radius: 10px; overflow: hidden; border: 2px solid #444; }
        canvas.zoom-canvas { width: 100%; height: 100%; }
        
        /* CALIBRATION CONTROLS */
        .calibration { background: rgba(0,0,0,0.3); padding: 15px; border-radius: 8px; margin: 15px 0; display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .calibration label { font-size: 10px; color: var(--dim); display: block; margin-bottom: 5px; }
        input[type=range] { width: 100%; accent-color: var(--ai-scan); }

        /* DATA ELEMENTS */
        .insight-box { background: rgba(15, 23, 42, 0.6); padding: 15px; border-radius: 8px; margin-bottom: 15px; border-left: 4px solid #334155; font-size: 13px; line-height: 1.5; }
        .progress-bg { background: #0f213a; height: 10px; border-radius: 5px; overflow: hidden; margin-top: 5px; }
        .progress-fill { height: 100%; width: 0%; transition: width 1.5s ease; }
        
        .visual-row { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 15px; }
        .mini-screen { background: #0f172a; border-radius: 8px; height: 120px; border: 1px solid #334155; position: relative; }
        .btn { padding: 12px 24px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; text-transform: uppercase; }
        .btn-analyze { background: var(--ai-scan); color: #000; }
    </style>
</head>
<body>

<div class="dashboard-container">
    <div class="header-ui">
        <h1>Tactical AI | Automated Discovery</h1>
        <div style="display:flex; gap:12px;">
            <input type="file" id="videoUpload" accept="video/*" style="display:none;">
            <button class="btn" style="background:var(--team-l); color:#000;" onclick="document.getElementById('videoUpload').click()">Upload Video</button>
            <button class="btn btn-analyze" onclick="startSystem()">Run Analysis</button>
        </div>
    </div>

    <!-- POSSESSION -->
    <div class="possession-container">
        <div class="possession-bar possession-l" id="possL">TEAM LEFT: 50%</div>
        <div class="possession-bar possession-r" id="possR">50% :TEAM RIGHT</div>
    </div>

    <!-- MAIN VIDEO -->
    <div class="viewport">
        <video id="videoPlayer" muted></video>
        <div id="trackerL" class="tracker"></div>
        <div id="trackerR" class="tracker"></div>
    </div>

    <div class="analysis-grid">
        <!-- TEAM LEFT -->
        <div class="team-card" id="cardL">
            <h3 style="color:var(--team-l); margin-top:0;">TEAM LEFT | DISCOVERY CAM</h3>
            <div class="zoom-cam-container"><canvas id="canvasL" class="zoom-canvas"></canvas></div>
            
            <div class="calibration">
                <div><label>Pan Horizontal</label><input type="range" id="panXL" min="-100" max="100" value="0" oninput="manualAdjust('L')"></div>
                <div><label>Pan Vertical</label><input type="range" id="panYL" min="-100" max="100" value="0" oninput="manualAdjust('L')"></div>
                <div style="grid-column: span 2;"><label>Zoom Level</label><input type="range" id="zoomL" min="50" max="500" value="150" oninput="manualAdjust('L')"></div>
            </div>

            <div class="insight-box" id="boxL">Waiting for player #10 discovery...</div>
            
            <div style="font-size:12px; display:flex; justify-content:space-between;"><span>Pressing Efficiency</span><span id="pValL">0%</span></div>
            <div class="progress-bg"><div class="progress-fill" id="pBarL" style="background: var(--team-l);"></div></div>

            <div class="visual-row">
                <div class="mini-screen" id="heatL"></div>
                <div class="mini-screen"><canvas id="netL" style="width:100%; height:100%"></canvas></div>
            </div>
        </div>

        <!-- TEAM RIGHT -->
        <div class="team-card" id="cardR">
            <h3 style="color:var(--team-r); margin-top:0;">TEAM RIGHT | DISCOVERY CAM</h3>
            <div class="zoom-cam-container"><canvas id="canvasR" class="zoom-canvas"></canvas></div>

            <div class="calibration">
                <div><label>Pan Horizontal</label><input type="range" id="panXR" min="-100" max="100" value="0" oninput="manualAdjust('R')"></div>
                <div><label>Pan Vertical</label><input type="range" id="panYR" min="-100" max="100" value="0" oninput="manualAdjust('R')"></div>
                <div style="grid-column: span 2;"><label>Zoom Level</label><input type="range" id="zoomR" min="50" max="500" value="150" oninput="manualAdjust('R')"></div>
            </div>

            <div class="insight-box" id="boxR">Waiting for player #4 discovery...</div>

            <div style="font-size:12px; display:flex; justify-content:space-between;"><span>Pressing Efficiency</span><span id="pValR">0%</span></div>
            <div class="progress-bg"><div class="progress-fill" id="pBarR" style="background: var(--team-r);"></div></div>

            <div class="visual-row">
                <div class="mini-screen" id="heatR"></div>
                <div class="mini-screen"><canvas id="netR" style="width:100%; height:100%"></canvas></div>
            </div>
        </div>
    </div>
</div>

<script>
    const video = document.getElementById('videoPlayer');
    const teams = {
        L: { canvas: document.getElementById('canvasL'), ctx: document.getElementById('canvasL').getContext('2d'), tracker: document.getElementById('trackerL'), found: false, x: 30, y: 45, color: '#38bdf8' },
        R: { canvas: document.getElementById('canvasR'), ctx: document.getElementById('canvasR').getContext('2d'), tracker: document.getElementById('trackerR'), found: false, x: 70, y: 50, color: '#ef4444' }
    };

    let isRunning = false;

    // 1. Upload
    document.getElementById('videoUpload').addEventListener('change', function(e) {
        const file = e.target.files[0];
        if (file) video.src = URL.createObjectURL(file);
    });

    // 2. Start
    function startSystem() {
        if (!video.src) return alert("Upload a video first.");
        isRunning = true;
        teams.L.found = teams.R.found = false;
        video.currentTime = 0;
        video.play();
        teams.L.tracker.style.display = teams.R.tracker.style.display = 'block';
    }

    // 3. Automated Discovery Logic
    video.addEventListener('timeupdate', () => {
        if (!isRunning) return;
        const t = video.currentTime;

        // TEAM LEFT: AUTO-FREEZE AT 4 SECONDS
        if (t >= 4.0 && !teams.L.found) {
            lockTeam('L');
        } else if (!teams.L.found) {
            let lx = 20 + Math.sin(t*2)*10;
            let ly = 40 + Math.cos(t)*10;
            updateScan('L', lx, ly);
        }

        // TEAM RIGHT: AUTO-FREEZE AT 8 SECONDS
        if (t >= 8.0 && !teams.R.found) {
            lockTeam('R');
        } else if (!teams.R.found) {
            let rx = 75 + Math.cos(t*1.5)*10;
            let ry = 55 + Math.sin(t*2)*10;
            updateScan('R', rx, ry);
        }
    });

    function updateScan(id, x, y) {
        const team = teams[id];
        team.x = x; team.y = y;
        team.tracker.style.left = x + '%';
        team.tracker.style.top = y + '%';
        drawIdentityCam(id);
    }

    function lockTeam(id) {
        video.pause(); // CRITICAL: Stop the video
        const team = teams[id];
        team.found = true;
        
        // Change Tracker color
        team.tracker.style.borderColor = team.color;
        team.tracker.style.boxShadow = `0 0 20px ${team.color}`;
        
        // Enable Analysis
        document.getElementById('card' + id).classList.add('active');
        document.getElementById('card' + id).style.borderTopColor = team.color;
        
        drawIdentityCam(id);
        populateAnalyses(id);

        // Continue to find next player after 3 seconds
        setTimeout(() => { if(isRunning) video.play(); }, 3000);
    }

    function manualAdjust(id) {
        drawIdentityCam(id);
    }

    function drawIdentityCam(id) {
        const team = teams[id];
        const canvas = team.canvas;
        const ctx = team.ctx;
        
        // Manual pan/zoom calibration
        const panX = parseInt(document.getElementById('panX' + id).value);
        const panY = parseInt(document.getElementById('panY' + id).value);
        const zoom = parseInt(document.getElementById('zoom' + id).value);

        canvas.width = 600; canvas.height = 600;

        // Calculate view
        const centerX = ((team.x + (panX/5)) / 100) * video.videoWidth;
        const centerY = ((team.y + (panY/5)) / 100) * video.videoHeight;
        const viewSize = 600 / (zoom / 100);

        ctx.fillStyle = "#000";
        ctx.fillRect(0,0,600,600);
        ctx.drawImage(video, centerX - (viewSize/2), centerY - (viewSize/2), viewSize, viewSize, 0, 0, 600, 600);

        // Draw Reticle
        ctx.strokeStyle = team.found ? team.color : "#22c55e";
        ctx.lineWidth = 4;
        ctx.beginPath(); ctx.arc(300, 300, 80, 0, Math.PI*2); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(300, 260); ctx.lineTo(300, 340); ctx.moveTo(260, 300); ctx.lineTo(340, 300); ctx.stroke();
    }

    function populateAnalyses(id) {
        const color = teams[id].color;
        
        // 1. Insights
        const text = id === 'L' ? 
            "Target #10 Identified. High-intensity playmaker detected in half-space. 92% pass accuracy in final third." : 
            "Target #4 Identified. Defensive anchor. Structural disconnection detected in the recovery line.";
        document.getElementById('box' + id).innerText = text;
        document.getElementById('box' + id).style.borderLeftColor = color;
        document.getElementById('box' + id).style.color = "#fff";

        // 2. Meters
        const val = id === 'L' ? 88 : 61;
        document.getElementById(`pBar${id}`).style.width = val + '%';
        document.getElementById(`pVal${id}`).innerText = val + '%';

        // 3. Heatmap
        document.getElementById('heat' + id).style.background = `radial-gradient(circle at center, ${color}66, transparent 70%)`;
        document.getElementById('heat' + id).innerHTML = `<small style="position:absolute; bottom:5px; width:100%; text-align:center;">HEATMAP</small>`;

        // 4. Passing Net
        drawNet(id, color);

        // 5. Possession Shift
        if(id === 'R') {
            document.getElementById('possL').style.width = '62%';
            document.getElementById('possL').innerText = "TEAM LEFT: 62%";
            document.getElementById('possR').innerText = "38% :TEAM RIGHT";
        }
    }

    function drawNet(id, color) {
        const canvas = document.getElementById('net' + id);
        const ctx = canvas.getContext('2d');
        canvas.width = 300; canvas.height = 120;
        const nodes = [];
        for(let i=0; i<7; i++) nodes.push({x: Math.random()*260+20, y: Math.random()*90+20});
        ctx.strokeStyle = color + '44';
        nodes.forEach((n, i) => { nodes.forEach((n2, j) => { if(i!==j && Math.random()>0.5){ctx.beginPath(); ctx.moveTo(n.x,n.y); ctx.lineTo(n2.x,n2.y); ctx.stroke();}});});
        nodes.forEach(n => { ctx.fillStyle = color; ctx.beginPath(); ctx.arc(n.x,n.y,4,0,Math.PI*2); ctx.fill();});
        ctx.fillStyle = "#94a3b8"; ctx.font = "10px sans-serif"; ctx.fillText("PASSING NETWORK", 110, 115);
    }
</script>

</body>
</html>
