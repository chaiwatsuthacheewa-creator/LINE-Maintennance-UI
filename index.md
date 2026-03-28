<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Smart Maintenance Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  :root { --bg-color: #0f172a; --card-bg: #1e293b; --card-border: #334155; --text-main: #f8fafc; --text-muted: #94a3b8; --primary: #3b82f6; --warning: #f59e0b; --success: #10b981; --danger: #ef4444; }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: 'Kanit', sans-serif; background-color: var(--bg-color); color: var(--text-main); padding: 20px; background-image: radial-gradient(circle at top right, rgba(30, 64, 175, 0.1), transparent 400px); }
  
  .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; padding: 0 10px; }
  .header-left { display: flex; align-items: center; gap: 16px; }
  .back-btn { display: flex; align-items: center; gap: 6px; color: var(--text-muted); text-decoration: none; padding: 8px 16px; border-radius: 8px; border: 1px solid var(--card-border); background: rgba(30, 41, 59, 0.5); transition: all 0.2s; font-size: 0.9rem; }
  .back-btn:hover { background: var(--card-bg); color: var(--text-main); border-color: var(--primary); }
  .header-left h2 { font-weight: 600; font-size: 1.5rem; letter-spacing: 0.5px; display: flex; align-items: center; gap: 12px;}
  .dot { width: 12px; height: 12px; border-radius: 50%; background: var(--success); box-shadow: 0 0 10px var(--success); animation: pulse 1.5s infinite; }
  @keyframes pulse { 0% {opacity: 1} 50% {opacity: 0.4} 100% {opacity: 1} }
  .header-right { display: flex; align-items: center; gap: 16px; color: var(--text-muted); font-weight: 300;}
  .time { text-align: right; }
  .time-big { font-size: 1.2rem; color: var(--text-main); font-weight: 500;}

  .dashboard-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; }
  .card { background: var(--card-bg); border: 1px solid var(--card-border); border-radius: 16px; padding: 20px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2); }
  .card-title { display: flex; align-items: center; gap: 8px; font-size: 1.1rem; margin-bottom: 16px; font-weight: 500; color: var(--text-main); }
  .span-2 { grid-column: span 2; }

  .kpi-card { display: flex; flex-direction: column; justify-content: space-between; position: relative; overflow: hidden; }
  .kpi-card::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 4px; }
  .kpi-total::before { background: var(--success); }
  .kpi-pending::before { background: var(--warning); }
  .kpi-done::before { background: var(--primary); }
  .kpi-sla::before { background: var(--danger); }
  .kpi-header { display: flex; justify-content: space-between; align-items: flex-start;}
  .kpi-icon-box { width: 48px; height: 48px; border-radius: 12px; display: flex; justify-content: center; align-items: center; font-size: 24px;}
  .kpi-total .kpi-icon-box { background: rgba(16, 185, 129, 0.1); color: var(--success); }
  .kpi-pending .kpi-icon-box { background: rgba(245, 158, 11, 0.1); color: var(--warning); }
  .kpi-done .kpi-icon-box { background: rgba(59, 130, 246, 0.1); color: var(--primary); }
  .kpi-sla .kpi-icon-box { background: rgba(239, 68, 68, 0.1); color: var(--danger); }
  .kpi-value { font-size: 2.5rem; font-weight: 600; margin-top: 10px;}
  .kpi-label { color: var(--text-muted); font-size: 0.9rem; }

  .building-container { display: flex; justify-content: space-around; align-items: flex-end; height: 100%; padding-bottom: 10px;}
  .building-item { text-align: center; display: flex; flex-direction: column; align-items: center; position: relative; cursor: pointer; transition: transform 0.2s; }
  .building-item:hover { transform: translateY(-5px); }
  .building-icon { margin-bottom: 12px; }
  .building-name { font-weight: 500; font-size: 0.9rem; }
  .building-jobs { font-size: 0.8rem; margin-top: 4px; padding: 2px 8px; border-radius: 4px; }
  .jobs-active { background: rgba(245, 158, 11, 0.2); color: var(--warning); }
  .jobs-clear { background: rgba(16, 185, 129, 0.2); color: var(--success); }
  
  .tooltip { visibility: hidden; position: absolute; bottom: 100%; left: 50%; transform: translateX(-50%); background: rgba(15, 23, 42, 0.95); border: 1px solid var(--card-border); padding: 10px 14px; border-radius: 8px; font-size: 0.85rem; width: max-content; z-index: 10; opacity: 0; transition: opacity 0.2s; box-shadow: 0 4px 15px rgba(0,0,0,0.5); text-align: left; }
  .building-item:hover .tooltip { visibility: visible; opacity: 1; }

  .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(15, 23, 42, 0.8); backdrop-filter: blur(5px); z-index: 1000; justify-content: center; align-items: center; opacity: 0; transition: opacity 0.3s; }
  .modal-overlay.show { display: flex; opacity: 1; }
  .modal { background: var(--card-bg); border: 1px solid var(--card-border); border-radius: 16px; padding: 24px; width: 90%; max-width: 550px; box-shadow: 0 10px 40px rgba(0,0,0,0.6); position: relative; transform: translateY(-20px); transition: transform 0.3s; }
  .modal-overlay.show .modal { transform: translateY(0); }
  .modal-close { position: absolute; top: 16px; right: 16px; background: none; border: none; color: var(--text-muted); font-size: 1.5rem; cursor: pointer; transition: color 0.2s; }
  .modal-close:hover { color: var(--danger); }
  .modal-title { font-size: 1.3rem; font-weight: 500; margin-bottom: 16px; display: flex; align-items: center; gap: 10px; color: var(--text-main); border-bottom: 1px solid var(--card-border); padding-bottom: 12px;}
  .modal-stats-row { display: flex; justify-content: space-between; margin-bottom: 16px; font-size: 0.9rem; }
  .modal-job-list { max-height: 250px; overflow-y: auto; padding-right: 8px; }
  .modal-job-list::-webkit-scrollbar { width: 6px; }
  .modal-job-list::-webkit-scrollbar-track { background: var(--card-bg); }
  .modal-job-list::-webkit-scrollbar-thumb { background: var(--card-border); border-radius: 4px; }
  .job-card { background: rgba(30, 41, 59, 0.5); border: 1px solid var(--card-border); border-radius: 8px; padding: 12px; margin-bottom: 8px; font-size: 0.85rem;}
  .job-card-header { display: flex; justify-content: space-between; margin-bottom: 6px; }

  .sla-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .sla-badge { background: rgba(239, 68, 68, 0.15); border: 1px solid rgba(239, 68, 68, 0.3); color: #fca5a5; padding: 10px 16px; border-radius: 99px; display: flex; align-items: center; gap: 8px; font-size: 0.85rem; }
  .sla-badge .alert-icon { background: var(--danger); color: white; width: 20px; height: 20px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 12px;}
  
  table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.9rem;}
  th { color: var(--text-muted); padding-bottom: 12px; font-weight: 400; border-bottom: 1px solid var(--card-border); }
  td { padding: 12px 0; border-bottom: 1px solid rgba(51, 65, 85, 0.5); }
  tr:last-child td { border-bottom: none; }
  .status-badge { padding: 4px 10px; border-radius: 6px; font-size: 0.8rem; font-weight: 500; }
  .status-done { background: rgba(59, 130, 246, 0.2); color: var(--primary); }
  .status-pending { background: rgba(245, 158, 11, 0.2); color: var(--warning); }
  
  #loadingState { position: fixed; top:0; left:0; width:100%; height:100%; background: var(--bg-color); z-index: 2000; display: flex; flex-direction: column; justify-content: center; align-items: center; color: var(--primary); font-size: 1.2rem; transition: opacity 0.3s;}

  @media (max-width: 1200px) { .dashboard-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (max-width: 768px) { .dashboard-grid { grid-template-columns: 1fr; } .span-2 { grid-column: span 1; } .sla-grid { grid-template-columns: 1fr; } }
</style>
</head>
<body>

<div id="loadingState">
  <div class="dot" style="width: 20px; height: 20px; margin-bottom: 10px;"></div>
  กำลังเชื่อมต่อข้อมูลจาก Google Sheets...
</div>

<div class="header">
  <div class="header-left">
    <a href="https://add00917-hue.github.io/GNR3-Building-MS/" class="back-btn">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M19 12H5M12 19l-7-7 7-7"/></svg>
      เมนูหลัก
    </a>
    <h2>🚀 Smart Maintenance Dashboard <div class="dot" style="display:inline-block; margin-left:8px;"></div></h2>
  </div>
  
  <div class="header-right">
    <div class="time" style="text-align: right;">
      <div class="time-big" id="timeEl">00:00 AM</div>
      <div id="dateEl">กำลังโหลด...</div>
      <div style="font-size: 0.75rem; color: var(--text-muted); margin-top: 4px; display: flex; align-items: center; justify-content: flex-end; gap: 6px;">
        <span id="syncStatus"><span class="dot" style="width: 8px; height: 8px; display: inline-block;"></span> กำลังซิงค์...</span>
        <button onclick="refreshData()" style="background: none; border: 1px solid var(--card-border); color: var(--primary); border-radius: 4px; padding: 2px 6px; cursor: pointer; font-size: 0.7rem; font-family: 'Kanit'; transition: all 0.2s;">อัปเดต</button>
      </div>
    </div>
    <div style="width:40px; height:40px; background:#334155; border-radius:50%; display:flex; justify-content:center; align-items:center; font-size:20px;">👤</div>
  </div>
</div>

<div class="dashboard-grid">
  <div class="card kpi-card kpi-total">
    <div class="kpi-header"><div><div class="kpi-label">งานทั้งหมด</div><div style="color:var(--text-muted); font-size:0.8rem;">งานรวมทั้งหมด</div></div><div class="kpi-icon-box">📋</div></div>
    <div class="kpi-value" style="color: var(--success)" id="total">-</div>
  </div>
  <div class="card kpi-card kpi-pending">
    <div class="kpi-header"><div><div class="kpi-label">งานค้าง</div><div style="color:var(--text-muted); font-size:0.8rem;">รอดำเนินการ</div></div><div class="kpi-icon-box">⏳</div></div>
    <div class="kpi-value" style="color: var(--warning)" id="open">-</div>
  </div>
  <div class="card kpi-card kpi-done">
    <div class="kpi-header"><div><div class="kpi-label">เสร็จแล้ว</div><div style="color:var(--text-muted); font-size:0.8rem;">ดำเนินการเสร็จสิ้น</div></div><div class="kpi-icon-box">✅</div></div>
    <div class="kpi-value" style="color: var(--primary)" id="complete">-</div>
  </div>
  <div class="card kpi-card kpi-sla">
    <div class="kpi-header"><div><div class="kpi-label">เกิน SLA</div><div style="color:var(--text-muted); font-size:0.8rem;">ต้องการดูแลด่วน</div></div><div class="kpi-icon-box">⚠️</div></div>
    <div class="kpi-value" style="color: var(--danger)" id="sla">-</div>
  </div>

  <div class="card span-2" style="display:flex; flex-direction:column;">
    <div class="card-title">🏢 สถานะอาคารและการค้างงาน <span style="font-size: 0.8rem; color: var(--text-muted); font-weight: 300; margin-left: auto;">(คลิกเพื่อดูรายละเอียด)</span></div>
    <div class="building-container" id="buildingContainer"></div>
  </div>

  <div class="card span-2">
    <div class="card-title">📈 Trend เปรียบเทียบงาน (เรียงตามวันที่รับงาน)</div>
    <div style="height: 200px; width: 100%;">
      <canvas id="trendChart"></canvas>
    </div>
  </div>

  <div class="card span-2">
    <div class="card-title" style="color: var(--danger);">⚠️ งานค้างเกิน 7 วัน (SLA)</div>
    <div class="sla-grid" id="slaContainer"></div>
  </div>

  <div class="card span-2">
    <div class="card-title">🛠️ งานล่าสุด (10 งาน)</div>
    <table>
      <thead>
        <tr><th>Ticket</th><th>อาคาร</th><th>รายละเอียด</th><th>Start</th><th>Status</th><th>Duration</th></tr>
      </thead>
      <tbody id="tableBody"></tbody>
    </table>
  </div>
</div>

<div class="modal-overlay" id="buildingModal">
  <div class="modal">
    <button class="modal-close" onclick="closeModal()">×</button>
    <div class="modal-title" id="modalTitle">🏢 ชื่ออาคาร</div>
    <div class="modal-stats-row">
      <div style="color: var(--success);">📋 รับงาน: <span id="modalTotal">0</span></div>
      <div style="color: var(--primary);">✅ เสร็จสิ้น: <span id="modalComplete">0</span></div>
      <div style="color: var(--warning);">⏳ งานค้าง: <span id="modalPending">0</span></div>
    </div>
    <div style="font-size: 0.9rem; margin-bottom: 8px; color: var(--text-muted);">รายการงานซ่อม:</div>
    <div class="modal-job-list" id="modalJobList"></div>
    <button onclick="closeModal()" style="width: 100%; padding: 10px; margin-top: 15px; border-radius: 8px; border: none; background: var(--card-border); color: var(--text-main); font-family: 'Kanit'; cursor: pointer; transition: background 0.2s;">ปิดหน้าต่าง</button>
  </div>
</div>

<script>
  // ==========================================
  // 🔴 1. Config
  // ==========================================
  const API_URL = "https://script.google.com/macros/s/AKfycbygiyeaAfjPr3dfl8JyFd_d-VeyuTfvlHl_PcJk4j-dZZY2lvCIOTC1SrZJ4yMvXau2GA/exec"; 
  const AUTO_REFRESH_MINUTES = 5; 

  // ==========================================
  // 🔵 2. Utility Functions
  // ==========================================
  function formatDuration(hours) {
    if (hours < 0) return '0 ชม.';
    if (hours >= 24) {
      const days = Math.floor(hours / 24);
      const remainHours = hours % 24;
      return remainHours > 0 ? `${days} วัน ${remainHours} ชม.` : `${days} วัน`;
    }
    return `${hours} ชม.`;
  }

  function parseCustomDate(dateStr) {
    if(!dateStr) return null;
    try {
      const parts = dateStr.split(' ');
      const dateParts = parts[0].includes('/') ? parts[0].split('/') : parts[0].split('-');
      const timeParts = parts[1] ? parts[1].split(':') : [0,0,0];
      return new Date(dateParts[2], parseInt(dateParts[1]) - 1, dateParts[0], timeParts[0], timeParts[1], timeParts[2] || 0);
    } catch(e) { 
      return new Date(dateStr); 
    }
  }

  function updateClock() {
    const now = new Date();
    document.getElementById('timeEl').innerText = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
    const days = ['อา.', 'จ.', 'อ.', 'พ.', 'พฤ.', 'ศ.', 'ส.'];
    document.getElementById('dateEl').innerText = `${days[now.getDay()]} ${now.getDate().toString().padStart(2,'0')}/${(now.getMonth()+1).toString().padStart(2,'0')}/${now.getFullYear().toString().slice(-2)}`;
  }
  setInterval(updateClock, 1000); updateClock();

  // ==========================================
  // 🟢 3. State Management
  // ==========================================
  const svgAri = `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="#60a5fa" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><path d="M6 22V4a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2v18M6 22h12M4 22h16M10 6h4M10 10h4M10 14h4M10 18h4"/></svg>`;
  const svgKSL = `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="#c084fc" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><path d="M8 22V7l4-4 4 4v15M4 22h16M12 7v15M9 11h.01M15 11h.01M9 15h.01M15 15h.01M9 19h.01M15 19h.01"/></svg>`;
  const svgESV = `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="#34d399" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 22V10a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v12M2 22h20M8 12h8M8 16h8M8 20h8"/></svg>`;
  const svgSaphan = `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="#fbbf24" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 22V12l9-7 9 7v10M2 22h20M9 14h6M9 18h6M12 14v8"/></svg>`;
  const svgRangsit = `<svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="#f87171" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 22V8h14v14M3 22h18M9 12h6M9 16h6M9 20h6"/></svg>`;

  let buildingMap = {}; 
  let mainData = []; 
  let myChart = null; 

  // ==========================================
  // 🟡 4. Fetch Data
  // ==========================================
  async function loadData(isSilent = false) {
    const syncEl = document.getElementById('syncStatus');
    const loadState = document.getElementById('loadingState');
    
    try {
      if (syncEl) syncEl.innerHTML = `<span class="dot" style="width: 8px; height: 8px; display: inline-block; background: var(--warning); box-shadow: 0 0 8px var(--warning);"></span> กำลังซิงค์...`;
      if (!isSilent && loadState) loadState.style.display = 'flex';

      const response = await fetch(API_URL);
      const rawData = await response.json();
      
      buildingMap = {
        ari: { name: 'Ari Hills', icon: svgAri, total: 0, pending: 0, completed: 0 },
        ksl: { name: 'KSL Tower', icon: svgKSL, total: 0, pending: 0, completed: 0 },
        esv: { name: 'ESV Tower', icon: svgESV, total: 0, pending: 0, completed: 0 },
        sph: { name: 'Saphan Mai', icon: svgSaphan, total: 0, pending: 0, completed: 0 },
        rng: { name: 'Rangsit', icon: svgRangsit, total: 0, pending: 0, completed: 0 }
      };

      mainData = [];

      rawData.forEach(row => {
        let rawBuilding = (row.Building || '').toLowerCase().trim();
        let bCode = 'ari'; 
        if (rawBuilding.includes('ari') || rawBuilding.includes('อารีย์')) bCode = 'ari';
        else if (rawBuilding.includes('ksl')) bCode = 'ksl';
        else if (rawBuilding.includes('esv')) bCode = 'esv';
        else if (rawBuilding.includes('saphan') || rawBuilding.includes('สะพาน')) bCode = 'sph';
        else if (rawBuilding.includes('rangsit') || rawBuilding.includes('รังสิต')) bCode = 'rng';

        const start = parseCustomDate(row.Date);
        const end = row.CloseTime ? parseCustomDate(row.CloseTime) : null;
        const status = (row.Status && row.Status.toLowerCase() === 'complete') ? 'completed' : 'pending';
        
        let durationInHours = 0;
        if(start) {
          const endTime = end ? end : new Date(); 
          durationInHours = Math.floor((endTime - start) / (1000 * 60 * 60));
        }

        mainData.push({
          ticket: row.Ticket || '-',
          buildingCode: bCode,
          detail: row.Detail || '-',
          start: start,
          duration: durationInHours,
          status: status,
          // ⚠️ อัปเดต SLA เป็น 168 ชั่วโมง (7 วัน)
          isOverSLA: durationInHours > 168 && status === 'pending'
        });

        buildingMap[bCode].total++;
        if(status === 'completed') buildingMap[bCode].completed++;
        else buildingMap[bCode].pending++;
      });

      renderDashboard(); 
      if (loadState) loadState.style.display = 'none'; 
      
      if (syncEl) {
        const now = new Date();
        const syncTime = `${now.getHours().toString().padStart(2,'0')}:${now.getMinutes().toString().padStart(2,'0')} น.`;
        syncEl.innerHTML = `<span class="dot" style="width: 8px; height: 8px; display: inline-block;"></span> ซิงค์ล่าสุด: ${syncTime}`;
      }
      
    } catch (error) {
      console.error('Error loading data:', error);
      if (!isSilent && loadState) {
        loadState.innerHTML = `<div style="color:var(--danger)">เกิดข้อผิดพลาดในการโหลดข้อมูล <br><small>โปรดรีเฟรชหน้าเว็บ หรือตรวจสอบ Google Sheets API</small></div>`;
      }
      if (syncEl) syncEl.innerHTML = `<span style="color: var(--danger);">❌ ซิงค์ล้มเหลว</span>`;
    }
  }

  // ==========================================
  // 🟠 5. Render UI
  // ==========================================
  function renderDashboard() {
    const keys = Object.keys(buildingMap);
    
    let globalTotal = mainData.length;
    let globalPending = mainData.filter(d => d.status === 'pending').length;
    let globalComplete = globalTotal - globalPending;
    const slaJobs = mainData.filter(d => d.isOverSLA).sort((a,b) => b.duration - a.duration); 

    document.getElementById('total').innerText = globalTotal;
    document.getElementById('open').innerText = globalPending;
    document.getElementById('complete').innerText = globalComplete;
    document.getElementById('sla').innerText = slaJobs.length;

    let buildingHTML = '';
    keys.forEach(k => {
      const b = buildingMap[k];
      const statusClass = b.pending > 0 ? 'jobs-active' : 'jobs-clear';
      const statusText = b.pending > 0 ? `${b.pending} งานค้าง` : `ไม่มีงานค้าง`;
      const tooltipHTML = `<div class="tooltip"><div style="color:var(--success)">รับงาน: ${b.total}</div><div style="color:var(--primary)">เสร็จสิ้น: ${b.completed}</div><div style="color:var(--warning)">งานค้าง: ${b.pending}</div></div>`;
      buildingHTML += `<div class="building-item" onclick="openModal('${k}')">${tooltipHTML}<div class="building-icon">${b.icon}</div><div class="building-name">${b.name}</div><div class="building-jobs ${statusClass}">${statusText}</div></div>`;
    });
    document.getElementById('buildingContainer').innerHTML = buildingHTML;

    let slaHTML = '';
    slaJobs.slice(0, 6).forEach(d => {
      slaHTML += `<div class="sla-badge"><div class="alert-icon">!</div>${d.ticket} | ${buildingMap[d.buildingCode].name} | ค้าง ${formatDuration(d.duration)}</div>`;
    });
    for(let i=slaJobs.length; i<6; i++) { slaHTML += `<div class="sla-badge" style="opacity:0.2; justify-content:center;">...</div>`; }
    document.getElementById('slaContainer').innerHTML = slaHTML;

    const latest = [...mainData].sort((a,b) => b.start - a.start).slice(0, 10); 
    let tableHTML = '';
    latest.forEach((d) => {
      const bName = buildingMap[d.buildingCode].name;
      const dateStr = d.start ? `${d.start.getHours().toString().padStart(2,'0')}:${d.start.getMinutes().toString().padStart(2,'0')} ${d.start.getDate().toString().padStart(2,'0')}/${(d.start.getMonth()+1).toString().padStart(2,'0')}/${d.start.getFullYear().toString().slice(-2)}` : '-';
      let statusHTML = d.status === 'completed' ? `<span class="status-badge status-done">✓ เสร็จแล้ว</span>` : `<span class="status-badge status-pending">⏳ รอดำเนินการ</span>`;
      tableHTML += `<tr><td>${d.ticket}</td><td>${bName}</td><td>${d.detail}</td><td style="color:var(--text-muted)">${dateStr}</td><td>${statusHTML}</td><td>${formatDuration(d.duration)}</td></tr>`;
    });
    document.getElementById('tableBody').innerHTML = tableHTML;

    renderChart();
  }

  // ==========================================
  // 🟣 6. Chart.js
  // ==========================================
  function renderChart() {
    const daysMap = {};
    mainData.forEach(d => {
      if(!d.start) return;
      const day = d.start.getDate(); 
      if(!daysMap[day]) daysMap[day] = { total: 0, completed: 0, pending: 0 };
      daysMap[day].total++;
      if(d.status === 'completed') daysMap[day].completed++;
      if(d.status === 'pending') daysMap[day].pending++;
    });

    const sortedDays = Object.keys(daysMap).map(Number).sort((a,b) => a-b);
    const labels = sortedDays.map(String);
    const totalData = sortedDays.map(d => daysMap[d].total);
    const completedData = sortedDays.map(d => daysMap[d].completed);
    const pendingData = sortedDays.map(d => daysMap[d].pending);

    const ctx = document.getElementById('trendChart').getContext('2d');
    if (myChart) myChart.destroy(); 

    myChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [
          { label: 'งานทั้งหมด', data: totalData, borderColor: '#10b981', backgroundColor: 'rgba(16, 185, 129, 0.1)', borderWidth: 2, tension: 0.4, fill: true, order: 3 },
          { label: 'เสร็จสิ้น', data: completedData, borderColor: '#3b82f6', borderWidth: 2, tension: 0.4, order: 2 },
          { label: 'งานค้าง', data: pendingData, borderColor: '#f59e0b', borderWidth: 2, borderDash: [5, 5], tension: 0.4, order: 1 }
        ]
      },
      options: {
        responsive: true, maintainAspectRatio: false,
        plugins: { legend: { position: 'top', labels: { color: '#94a3b8', boxWidth: 12, usePointStyle: true, font: {family: 'Kanit'} } }, tooltip: { titleFont: {family: 'Kanit'}, bodyFont: {family: 'Kanit'} } },
        scales: {
          x: { title: { display: true, text: 'วันที่รับงาน', color: '#94a3b8', font: {family: 'Kanit'} }, grid: { color: 'rgba(51, 65, 85, 0.4)' }, ticks: { color: '#94a3b8', font: {family: 'Kanit'} } },
          y: { title: { display: true, text: 'จำนวนงาน', color: '#94a3b8', font: {family: 'Kanit'} }, grid: { color: 'rgba(51, 65, 85, 0.4)' }, ticks: { color: '#94a3b8', font: {family: 'Kanit'} }, beginAtZero: true }
        },
        elements: { point: { radius: 4, hoverRadius: 6 } }, interaction: { mode: 'index', intersect: false }
      }
    });
  }

  // ==========================================
  // 🟤 7. Modal & Refresh
  // ==========================================
  const modalOverlay = document.getElementById('buildingModal');

  function openModal(buildingKey) {
    const b = buildingMap[buildingKey];
    document.getElementById('modalTitle').innerHTML = `${b.icon} ${b.name}`;
    document.getElementById('modalTotal').innerText = b.total;
    document.getElementById('modalComplete').innerText = b.completed;
    document.getElementById('modalPending').innerText = b.pending;
    
    const bData = mainData.filter(d => d.buildingCode === buildingKey).sort((a,b) => b.start - a.start);
    let jobsHTML = '';
    
    if(bData.length === 0) {
      jobsHTML = '<div style="text-align:center; padding: 20px; color: var(--text-muted);">ไม่มีประวัติการแจ้งซ่อม</div>';
    } else {
      bData.forEach(job => {
        const badge = job.status === 'completed' ? `<span class="status-badge status-done">✓ เสร็จสิ้น</span>` : `<span class="status-badge status-pending">⏳ งานค้าง</span>`;
        const dateStr = job.start ? `${job.start.getDate().toString().padStart(2,'0')}/${(job.start.getMonth()+1).toString().padStart(2,'0')}/${job.start.getFullYear().toString().slice(-2)}` : '-';
        jobsHTML += `<div class="job-card"><div class="job-card-header"><span style="font-weight: 500; color: var(--text-main);">${job.ticket} - ${job.detail}</span>${badge}</div><div style="display: flex; justify-content: space-between; color: var(--text-muted);"><span>📅 ${dateStr}</span><span>⏱ ใช้เวลา / ค้างมาแล้ว ${formatDuration(job.duration)}</span></div></div>`;
      });
    }
    document.getElementById('modalJobList').innerHTML = jobsHTML;
    modalOverlay.classList.add('show');
  }

  function closeModal() { 
    modalOverlay.classList.remove('show'); 
  }

  modalOverlay.addEventListener('click', function(e) { 
    if(e.target === modalOverlay) closeModal(); 
  });

  function refreshData() {
    const loadState = document.getElementById('loadingState');
    if(loadState) {
      loadState.style.display = 'flex';
      loadState.innerHTML = `<div class="dot" style="width: 20px; height: 20px; margin-bottom: 10px;"></div>กำลังดึงข้อมูลล่าสุด...`;
    }
    loadData(false); 
  }

  setInterval(() => {
    loadData(true); 
  }, AUTO_REFRESH_MINUTES * 60 * 1000);

  loadData();
</script>
</body>
</html>
