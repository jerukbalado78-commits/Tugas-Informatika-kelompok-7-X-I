<!doctype html>

<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Presensi Manual & Keterlambatan</title>
  <style>
    :root{--bg:#f7f8fb;--card:#ffffff;--muted:#666}
    body{font-family:Inter, system-ui, Arial; background:var(--bg); margin:0;padding:24px}
    .container{max-width:980px;margin:0 auto}
    .card{background:var(--card);padding:18px;border-radius:12px;box-shadow:0 6px 18px rgba(10,10,20,0.06);}
    h1{margin:0 0 8px;font-size:20px}
    p.small{color:var(--muted);margin:0 0 12px}
    form{display:grid;grid-template-columns:1fr 120px;gap:12px;align-items:end}
    label{display:block;font-size:13px;color:#222;margin-bottom:6px}
    input,select,button{width:100%;padding:10px;border-radius:8px;border:1px solid #e2e6ef;font-size:14px}
    button.primary{background:#2b7cff;color:#fff;border:0}
    .controls{display:flex;gap:8px;margin-top:12px}
    table{width:100%;border-collapse:collapse;margin-top:16px}
    th,td{padding:10px;border-bottom:1px solid #eef2fb;text-align:left;font-size:14px}
    th{background:#fafcff}
    .badge{display:inline-block;padding:4px 8px;border-radius:999px;font-size:12px}
    .on-time{background:#e6f7ea;color:#0a7a2b}
    .late{background:#fff3ee;color:#b02a1b}
    .muted{color:#777;font-size:13px}
    .smallnote{font-size:13px;color:#444;margin-top:8px}
    .topbar{display:flex;justify-content:space-between;gap:12px;align-items:center;margin-bottom:12px}
    .flexcol{display:flex;gap:8px}
    @media(max-width:640px){form{grid-template-columns:1fr;}}
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <div class="topbar">
        <div>
          <h1>Form Presensi Manual</h1>
          <p class="small">Isi nama, NIS / ID, dan waktu kedatangan. Sistem akan menandai otomatis jika terlambat.</p>
        </div>
        <div class="flexcol">
          <label for="threshold">Batas Waktu (Jam:Menit)</label>
          <input id="threshold" value="07:00" title="Format HH:MM" />
        </div>
      </div><form id="attendanceForm" onsubmit="return false;">
    <div>
      <label for="name">Nama</label>
      <input id="name" placeholder="Nama lengkap" required />
    </div>

    <div>
      <label for="idnum">NIS / ID</label>
      <input id="idnum" placeholder="Nomer induk" />
    </div>

    <div>
      <label for="time">Waktu Kedatangan</label>
      <input id="time" type="time" />
    </div>

    <div>
      <label for="manualLate">Tandai Keterlambatan Manual</label>
      <select id="manualLate">
        <option value="auto">Otomatis</option>
        <option value="on">Terlambat</option>
        <option value="off">Tidak Terlambat</option>
      </select>
    </div>

    <div style="grid-column:1/-1;display:flex;gap:8px;margin-top:6px">
      <button class="primary" id="checkinBtn">Check-in</button>
      <button id="nowBtn" type="button">Set ke Sekarang</button>
      <button id="clearForm" type="button">Bersihkan</button>
    </div>
  </form>

  <div class="smallnote">Catatan: "Otomatis" menggunakan <strong>batas waktu</strong> di atas untuk menentukan terlambat.</div>

  <div style="margin-top:18px">
    <div style="display:flex;gap:8px;align-items:center">
      <h3 style="margin:0">Daftar Presensi</h3>
      <div class="muted">(Data disimpan di penyimpanan browser)</div>
    </div>

    <div class="controls">
      <button id="exportCsv">Export CSV</button>
      <button id="loadSample">Isi Contoh</button>
      <button id="clearAll">Hapus Semua</button>
    </div>

    <table id="table">
      <thead>
        <tr><th>#</th><th>Nama</th><th>ID</th><th>Waktu</th><th>Status</th><th>Aksi</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>
</div>

  </div>  <script>
    // Simple attendance app: presensi manual dengan deteksi keterlambatan.
    const STORAGE_KEY = 'presensi_manual_data_v1';
    const form = document.getElementById('attendanceForm');
    const nameInput = document.getElementById('name');
    const idInput = document.getElementById('idnum');
    const timeInput = document.getElementById('time');
    const checkinBtn = document.getElementById('checkinBtn');
    const nowBtn = document.getElementById('nowBtn');
    const tableBody = document.querySelector('#table tbody');
    const thresholdInput = document.getElementById('threshold');
    const manualLate = document.getElementById('manualLate');

    function loadData(){
      try{
        const raw = localStorage.getItem(STORAGE_KEY);
        return raw ? JSON.parse(raw) : [];
      }catch(e){return []}
    }
    function saveData(arr){ localStorage.setItem(STORAGE_KEY, JSON.stringify(arr)); }

    function formatTime(t){
      if(!t) return '';
      // t could be HH:MM
      return t;
    }

    function isLate(arrival, threshold){
      // arrival, threshold = "HH:MM"
      if(!arrival) return false;
      const [ah,am] = arrival.split(':').map(Number);
      const [th,tm] = threshold.split(':').map(Number);
      if(ah > th) return true;
      if(ah === th && am > tm) return true;
      return false;
    }

    function render(){
      const data = loadData();
      tableBody.innerHTML = '';
      data.forEach((row, idx) => {
        const tr = document.createElement('tr');
        const statusBadge = document.createElement('span');
        statusBadge.className = 'badge ' + (row.late ? 'late' : 'on-time');
        statusBadge.textContent = row.late ? 'Terlambat' : 'Tepat Waktu';

        tr.innerHTML = `
          <td>${idx+1}</td>
          <td>${escapeHtml(row.name)}</td>
          <td>${escapeHtml(row.id||'')}</td>
          <td>${escapeHtml(row.time)}</td>
          <td></td>
          <td>
            <button data-idx="${idx}" class="editBtn">Edit</button>
            <button data-idx="${idx}" class="delBtn">Hapus</button>
          </td>
        `;
        tr.children[4].appendChild(statusBadge);
        tableBody.appendChild(tr);
      });
      attachRowListeners();
    }

    function escapeHtml(s){ return String(s||'').replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }

    function attachRowListeners(){
      document.querySelectorAll('.delBtn').forEach(b=>{
        b.onclick = e=>{
          const idx = Number(e.target.dataset.idx);
          const arr = loadData(); arr.splice(idx,1); saveData(arr); render();
        }
      });
      document.querySelectorAll('.editBtn').forEach(b=>{
        b.onclick = e=>{
          const idx = Number(e.target.dataset.idx);
          const arr = loadData();
          const r = arr[idx];
          if(!r) return;
          nameInput.value = r.name;
          idInput.value = r.id||'';
          timeInput.value = r.time;
          manualLate.value = r.manualStatus || 'auto';
          // remove old and re-add when checking in
          arr.splice(idx,1); saveData(arr); render();
        }
      });
    }

    // set time input to now (HH:MM)
    function setTimeNow(){
      const d = new Date();
      const hh = String(d.getHours()).padStart(2,'0');
      const mm = String(d.getMinutes()).padStart(2,'0');
      timeInput.value = hh + ':' + mm;
    }

    nowBtn.onclick = ()=> setTimeNow();

    checkinBtn.onclick = ()=>{
      const name = nameInput.value.trim();
      if(!name){ alert('Nama wajib diisi'); nameInput.focus(); return; }
      const id = idInput.value.trim();
      const time = timeInput.value || (new Date().toTimeString().slice(0,5));
      const threshold = thresholdInput.value || '07:00';
      let late = false;
      if(manualLate.value === 'on') late = true;
      else if(manualLate.value === 'off') late = false;
      else late = isLate(time, threshold);

      const arr = loadData();
      arr.push({name, id, time, late, manualStatus: manualLate.value});
      saveData(arr); render();
      form.reset();
    }

    document.getElementById('exportCsv').onclick = ()=>{
      const arr = loadData();
      if(!arr.length){ alert('Belum ada data untuk diekspor'); return; }
      const header = ['Nama','ID','Waktu','Status'];
      const rows = arr.map(r => [r.name, r.id||'', r.time, r.late ? 'Terlambat' : 'Tepat Waktu']);
      const csv = [header, ...rows].map(r=> r.map(c=> '"'+String(c).replaceAll('"','""')+'"').join(',')).join('\n');
      const blob = new Blob([csv], {type:'text/csv'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'presensi.csv'; a.click(); URL.revokeObjectURL(url);
    }

    document.getElementById('clearAll').onclick = ()=>{
      if(confirm('Hapus semua data presensi dari browser?')){ localStorage.removeItem(STORAGE_KEY); render(); }
    }

    document.getElementById('loadSample').onclick = ()=>{
      const sample = [
        {name:'Ani Wijaya', id:'101', time:'06:50', late:false, manualStatus:'auto'},
        {name:'Budi Santoso', id:'102', time:'07:15', late:true, manualStatus:'auto'},
      ];
      saveData(sample); render();
    }

    document.getElementById('clearForm').onclick = ()=> form.reset();

    // init
    render();
    // prefill time with now for convenience
    setTimeNow();
  </script></body>
</html>
