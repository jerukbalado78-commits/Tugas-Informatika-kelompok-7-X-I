<!DOCTYPE html><html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Presensi Manual & Keterlambatan</title>
  <style>
    body {font-family: Arial, sans-serif; background: #f4f6f9; margin: 0; padding: 20px;}
    .container {max-width: 800px; margin: auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 3px 8px rgba(0,0,0,0.1);}
    h2 {text-align: center;}
    form {display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px;}
    label {font-weight: bold;}
    input, button {padding: 8px; border: 1px solid #ccc; border-radius: 6px;}
    button {cursor: pointer;}
    table {width: 100%; border-collapse: collapse; margin-top: 20px;}
    th, td {border: 1px solid #ddd; padding: 10px; text-align: center;}
    th {background: #f0f0f0;}
    .on-time {background: #d4edda; color: #155724; padding: 5px; border-radius: 5px;}
    .late {background: #f8d7da; color: #721c24; padding: 5px; border-radius: 5px;}
  </style>
</head>
<body>
  <div class="container">
    <h2>Presensi Manual & Deteksi Keterlambatan</h2>
    <form id="presensiForm">
      <div>
        <label for="nama">Nama:</label>
        <input type="text" id="nama" required>
      </div>
      <div>
        <label for="nis">NIS / ID:</label>
        <input type="text" id="nis">
      </div>
      <div>
        <label for="waktu">Waktu Kehadiran:</label>
        <input type="time" id="waktu">
      </div>
      <div>
        <label for="batas">Batas Waktu:</label>
        <input type="time" id="batas" value="07:00">
      </div>
      <div style="grid-column: span 2; text-align:center;">
        <button type="submit">Check-in</button>
      </div>
    </form><table>
  <thead>
    <tr>
      <th>No</th>
      <th>Nama</th>
      <th>NIS / ID</th>
      <th>Waktu</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody id="dataBody"></tbody>
</table>

  </div>  <script>
    const form = document.getElementById('presensiForm');
    const tbody = document.getElementById('dataBody');
    let counter = 0;

    form.addEventListener('submit', function(e){
      e.preventDefault();

      const nama = document.getElementById('nama').value;
      const nis = document.getElementById('nis').value;
      const waktu = document.getElementById('waktu').value || new Date().toTimeString().slice(0,5);
      const batas = document.getElementById('batas').value;

      counter++;

      // cek terlambat
      const status = cekTerlambat(waktu, batas) ? '<span class="late">Terlambat</span>' : '<span class="on-time">Tepat Waktu</span>';

      const row = `<tr>
        <td>${counter}</td>
        <td>${nama}</td>
        <td>${nis}</td>
        <td>${waktu}</td>
        <td>${status}</td>
      </tr>`;

      tbody.innerHTML += row;
      form.reset();
    });

    function cekTerlambat(waktu, batas){
      const [h1, m1] = waktu.split(':').map(Number);
      const [h2, m2] = batas.split(':').map(Number);
      if(h1 > h2) return true;
      if(h1 === h2 && m1 > m2) return true;
      return false;
    }
  </script></body>
</html>
