<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Smart Maintenance Dashboard</title>

<link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
  :root {
    --bg-color: #0f172a; /* Slate 900 */
    --card-bg: #1e293b; /* Slate 800 */
    --card-border: #334155;
    --text-main: #f8fafc;
    --text-muted: #94a3b8;
    
    --primary: #3b82f6; /* Blue */
    --warning: #f59e0b; /* Orange */
    --success: #10b981; /* Green */
    --danger: #ef4444;  /* Red */
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'Kanit', sans-serif;
    background-color: var(--bg-color);
    color: var(--text-main);
    padding: 20px;
    background-image: radial-gradient(circle at top right, rgba(30, 64, 175, 0.1), transparent 400px);
  }

  /* Header */
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 24px;
    padding: 0 10px;
  }
  .header-left { display: flex; align-items: center; gap: 12px; }
  .header-left h2 { font-weight: 600; font-size: 1.5rem; letter-spacing: 0.5px; }
  .dot { width: 12px; height: 12px; border-radius: 50%; background: var(--success); box-shadow: 0 0 10px var(--success); animation: pulse 1.5s infinite; }
  
  .header-right { display: flex; align-items: center; gap: 16px; color: var(--text-muted); font-weight: 300;}
  .header-right .time { text-align: right; }
  .header-right .time-big { font-size: 1.2rem; color: var(--text-main); font-weight: 500;}
  
  @keyframes pulse { 0% {opacity: 1} 50% {opacity: 0.4} 100% {opacity: 1} }

  /* Grid Layout */
  .dashboard-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
  }

  /* Cards */
  .card {
    background: var(--card-bg);
    border: 1px solid var(--card-border);
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  }
  .card-title { display: flex; align-items: center; gap: 8px; font-size: 1.1rem; margin-bottom: 16px; font-weight: 500; color: var(--text-main); }
  .card-title svg { width: 20px; height: 20px; }

  /* KPI Cards */
  .kpi-card { display: flex; flex-direction: column; justify-content: space-between; position: relative; overflow: hidden; }
  .kpi-card::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 4px; }
  .kpi-total::before { background: var(--primary); }
  .kpi-pending::before { background: var(--warning); }
  .kpi-done::before { background: var(--success); }
  .kpi-sla::before { background: var(--danger); }

  .kpi-header { display: flex; justify-content: space-between; align-items: flex-start;}
  .kpi-icon-box { width: 48px; height: 48px; border-radius: 12px; display: flex; justify-content: center; align-items: center; font-size: 24px;}
  .kpi-total .kpi-icon-box { background: rgba(59, 130, 246, 0.1); color: var(--primary); }
  .kpi-pending .kpi-icon-box { background: rgba(245, 158, 11, 0.1); color: var(--warning); }
  .kpi-done .kpi-icon-box { background: rgba(16, 185, 129, 0.1); color: var(--success); }
  .kpi-sla .kpi-icon-box { background: rgba(239, 68, 68, 0.1); color: var(--danger); }

  .kpi-value { font-size: 2.5rem; font-weight: 600; margin-top: 10px;}
  .kpi-label { color: var(--text-muted); font-size: 0.9rem; }

  /* Sections */
  .span-2 { grid-column: span 2; }
  
  /* Building Stats */
  .building-container { display: flex; justify-content: space-around; align-items: flex-end; height: 100%; padding-bottom: 20px;}
  .building-item { text-align: center; }
  .building-icon { font-size: 40px; margin-bottom: 8px; }
  .building-name { font-weight: 500; font-size: 0.9rem; }
  .building-jobs { color: var(--text-muted); font-size: 0.8rem; }

  /* SLA List */
  .sla-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .sla-badge { 
    background: rgba(239, 68, 68, 0.15); 
    border: 1px solid rgba(239, 68, 68, 0.3);
    color: #fca5a5;
    padding: 10px 16px;
    border-radius: 99px;
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.85rem;
  }
  .sla-badge .alert-icon { background: var(--danger); color: white; width: 20px; height: 20px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 12px;}

  /* Table */
  table { width: 100%; border-collapse: collapse; text-align: left; font-size: 0.9rem;}
  th { color: var(--text-muted); padding-bottom: 12px; font-weight: 400; border-bottom: 1px solid var(--card-border); }
  td { padding: 12px 0; border-bottom: 1px solid rgba(51, 65, 85, 0.5); }
  tr:last-child td { border-bottom: none; }
  
  .status-badge { padding: 4px 10px; border-radius: 6px; font-size: 0.8rem; font-weight: 500; }
  .status-done { background: rgba(16, 185, 129, 0.2); color: var(--success); }
  .status-fast { border: 1px solid var(--success); color: var(--success); }

  /* Responsive */
  @media (max-width: 1200px) { .dashboard-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (max-width: 768px) { .dashboard-grid { grid-template-columns: 1fr; } .span-2 { grid-column: span 1; } .sla-grid { grid-template-columns: 1fr; } }
</style>
</head>
<body>

<div class="header">
  <div class="header-left">
    <h2>🚀 Smart Maintenance Dashboard</h2>
    <div class="dot"></div>
  </div>
  <div class="header-right">
    <div class="time">
      <div class="time-big" id="timeEl">09:09 AM</div>
      <div id="dateEl">อ. 20/12/26</div>
    </div>
    <div style="width:40px; height:40px; background:#334155; border-radius:50%; display:flex; justify-content:center; align-items:center; font-size:20px;">👤</div>
  </div>
</div>

<div class="dashboard-grid">

  <div class="card kpi-card kpi-total">
    <div class="kpi-header">
      <div>
        <div class="kpi-label">งานทั้งหมด</div>
        <div style="color:var(--text-muted); font-size:0.8rem;">งานรวม 40 งาน</div>
      </div>
      <div class="kpi-icon-box">📋</div>
    </div>
    <div class="kpi-value" id="total">40</div>
  </div>

  <div class="card kpi-card kpi-pending">
    <div class="kpi-header">
      <div>
        <div class="kpi-label">งานค้าง</div>
        <div style="color:var(--text-muted); font-size:0.8rem;">รอดำเนินการ</div>
      </div>
      <div class="kpi-icon-box">⏳</div>
    </div>
    <div class="kpi-value" style="color: var(--warning)" id="open">15</div>
  </div>

  <div class="card kpi-card kpi-done">
    <div class="kpi-header">
      <div>
        <div class="kpi-label">เสร็จแล้ว</div>
        <div style="color:var(--text-muted); font-size:0.8rem;">ดำเนินการเสร็จสิ้น</div>
      </div>
      <div class="kpi-icon-box">✅</div>
    </div>
    <div class="kpi-value" style="color: var(--success)" id="complete">25</div>
  </div>

  <div class="card kpi-card kpi-sla">
    <div class="kpi-header">
      <div>
        <div class="kpi-label">เกิน SLA</div>
        <div style="color:var(--text-muted); font-size:0.8rem;">ต้องการดูแลด่วน</div>
      </div>
      <div class="kpi-icon-box">⚠️</div>
    </div>
    <div class="kpi-value" style="color: var(--danger)" id="sla">5</div>
  </div>

  <div class="card span-2" style="display:flex; flex-direction:column;">
    <div class="card-title">🏢 อาคารทั้งหมด</div>
    <div class="building-container" id="buildingContainer">
      </div>
  </div>

  <div class="card span-2">
    <div class="card-title">📈 Workload Trend</div>
    <div style="height: 200px; width: 100%;">
      <canvas id="trendChart"></canvas>
    </div>
  </div>

  <div class="card span-2">
    <div class="card-title" style="color: var(--danger);">⚠️ งานเกิน SLA</div>
    <div class="sla-grid" id="slaContainer">
      </div>
  </div>

  <div class="card span-2">
    <div class="card-title">🛠️ งานล่าสุด (10 งาน)</div>
    <table>
      <thead>
        <tr>
          <th>Ticket</th>
          <th>อาคาร</th>
          <th>รายละเอียด</th>
          <th>Start</th>
          <th>Status</th>
          <th>Duration</th>
        </tr>
      </thead>
      <tbody id="tableBody">
        </tbody>
    </table>
  </div>

</div>

<script>
  // Clock Logic
  const timeEl = document.getElementById('timeEl');
  const dateEl = document.getElementById('dateEl');
  
  function updateClock() {
    const now = new Date();
    timeEl.innerText = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
    const days = ['อา.', 'จ.', 'อ.', 'พ.', 'พฤ.', 'ศ.', 'ส.'];
    const dateStr = `${days[now.getDay()]} ${now.getDate().toString().padStart(2,'0')}/${(now.getMonth()+1).toString().padStart(2,'0')}/${now.getFullYear().toString().slice(-2)}`;
    dateEl.innerText = dateStr;
  }
  setInterval(updateClock, 1000);
  updateClock();

  // Data Generation
  const buildingMap = {
    ari: { name: 'Ari Hills', icon: '🏢' },
    ksl: { name: 'KSL Tower', icon: '🏥' },
    sph: { name: 'Saphan Mai Center', icon: '🏫' },
    tin: { name: 'Tin Center', icon: '🏬' }
  };
  const keys = Object.keys(buildingMap);
  const issues = ['แอร์ไม่เย็น', 'ไฟดับ', 'น้ำรั่ว', 'ระบบขัดข้อง'];
  const data = [];

  for(let i=1; i<=40; i++){
    const start = new Date(2026, 11, Math.floor(Math.random()*20)+1, 15, 30);
    const duration = Math.floor(Math.random()*100) + 1;
    data.push({
      ticket: 'NTL' + i.toString().padStart(2,'0'),
      buildingCode: keys[Math.floor(Math.random()*keys.length)],
      detail: issues[Math.floor(Math.random()*4)],
      start: start,
      duration: duration,
      isOverSLA: duration > 50 && Math.random() > 0.5 // สุ่มให้บางอันเกิน SLA
    });
  }

  // KPIs
  const total = data.length;
  const complete = 25;
  const open = total - complete;
  const slaJobs = data.filter(d => d.isOverSLA).slice(0, 5); // จำลองว่ามี 5 งานเกิน SLA
  
  document.getElementById('total').innerText = total;
  document.getElementById('open').innerText = open;
  document.getElementById('complete').innerText = complete;
  document.getElementById('sla').innerText = slaJobs.length;

  // Buildings Generate
  let buildingHTML = '';
  keys.forEach(k => {
    const activeJobs = Math.floor(Math.random() * 8) + 1;
    buildingHTML += `
      <div class="building-item">
        <div class="building-icon">${buildingMap[k].icon}</div>
        <div class="building-name">${buildingMap[k].name}</div>
        <div class="building-jobs">${activeJobs} active jobs</div>
      </div>
    `;
  });
  document.getElementById('buildingContainer').innerHTML = buildingHTML;

  // SLA Generate
  let slaHTML = '';
  slaJobs.forEach(d => {
    const daysOver = Math.floor(d.duration / 24);
    slaHTML += `
      <div class="sla-badge">
        <div class="alert-icon">!</div>
        ${d.ticket} | ${buildingMap[d.buildingCode].name} | ${d.detail} | เกิน ${daysOver} วัน
      </div>
    `;
  });
  // Fill empty slots if less than 6
  for(let i=slaJobs.length; i<6; i++) {
    slaHTML += `<div class="sla-badge" style="opacity:0.3; justify-content:center;">...</div>`;
  }
  document.getElementById('slaContainer').innerHTML = slaHTML;

  // Table Generate (Latest 10)
  const latest = data.slice(0, 6); // แสดง 6 บรรทัดให้พอดีการ์ด
  let tableHTML = '';
  const fastestDuration = Math.min(...latest.map(d => d.duration));

  latest.forEach(d => {
    const bName = buildingMap[d.buildingCode].name;
    const dateStr = `${d.start.getHours()}:${d.start.getMinutes()} ${d.start.getDate()}/${d.start.getMonth()+1}/${d.start.getFullYear().toString().slice(-2)}`;
    
    let statusHTML = `<span class="status-badge status-done">✓ เสร็จแล้ว</span>`;
    if(d.duration === fastestDuration) {
       statusHTML = `<span class="status-badge status-fast">Fast</span>`;
    }

    tableHTML += `
      <tr>
        <td>${d.ticket}</td>
        <td>${bName}</td>
        <td>${d.detail}</td>
        <td style="color:var(--text-muted)">${dateStr}</td>
        <td>${statusHTML}</td>
        <td>${d.duration} ชม</td>
      </tr>
    `;
  });
  document.getElementById('tableBody').innerHTML = tableHTML;

  // Chart.js Setup
  const ctx = document.getElementById('trendChart').getContext('2d');
  new Chart(ctx, {
    type: 'line',
    data: {
      labels: ['10', '12', '15', '17', '20', '22', '25'],
      datasets: [
        {
          label: 'Workload',
          data: [5, 12, 19, 15, 8, 14, 10],
          borderColor: '#10b981', /* Green */
          backgroundColor: 'rgba(16, 185, 129, 0.1)',
          borderWidth: 2,
          tension: 0.4,
          fill: true
        },
        {
          label: 'งานค้าง',
          data: [2, 8, 12, 6, 20, 15, 4],
          borderColor: '#f59e0b', /* Orange */
          borderWidth: 2,
          tension: 0.4
        },
        {
          label: 'Workstatus',
          data: [9, 5, 4, 22, 18, 6, 15],
          borderColor: '#3b82f6', /* Blue */
          borderWidth: 2,
          tension: 0.4
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'top',
          labels: { color: '#94a3b8', boxWidth: 12, usePointStyle: true }
        }
      },
      scales: {
        x: { grid: { color: 'rgba(51, 65, 85, 0.5)' }, ticks: { color: '#94a3b8' } },
        y: { grid: { color: 'rgba(51, 65, 85, 0.5)' }, ticks: { color: '#94a3b8' }, beginAtZero: true }
      },
      elements: { point: { radius: 3, hoverRadius: 6 } }
    }
  });
</script>
</body>
</html>
