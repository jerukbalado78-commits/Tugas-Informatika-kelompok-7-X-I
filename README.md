<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Presensi Manual</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: #f9f9f9;
    }
    h2 {
      color: #333;
    }
    form {
      margin-bottom: 20px;
    }
    input, button {
      padding: 8px;
      margin: 5px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
    }
    table, th, td {
      border: 1px solid #aaa;
    }
    th, td {
      padding: 10px;
      text-align: center;
    }
    .tepat {
      color: green;
      font-weight: bold;
    }
    .terlambat {
      color: red;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h2>📋 Sistem Presensi Manual</h2>

  <form id="presensiForm">
    <label>Nama: </label>
    <input type="text" id="nama" required>
    <label>Jam Masuk: </label>
    <input type="time" id="jamMasuk" required>
    <button type="submit">Catat Presensi</button>
  </form>

  <table>
    <thead>
      <tr>
        <th>Nama</th>
        <th>Jam Masuk</th>
        <th>Status</th>
      </tr>
    </thead>
    <tbody id="rekapPresensi">
    </tbody>
  </table>

  <script>
    const batasJamMasuk = "08:00";
    const rekapPresensi = document.getElementById("rekapPresensi");

    document.getElementById("presensiForm").addEventListener("submit", function(e) {
      e.preventDefault();

      const nama = document.getElementById("nama").value;
      const jamMasuk = document.getElementById("jamMasuk").value;

      let status = "";
      if (jamMasuk <= batasJamMasuk) {
        status = "<span class='tepat'>Tepat Waktu</span>";
      } else {
        status = "<span class='terlambat'>Terlambat</span>";
      }

      const row = `
        <tr>
          <td>${nama}</td>
          <td>${jamMasuk}</td>
          <td>${status}</td>
        </tr>
      `;

      rekapPresensi.innerHTML += row;

      // Reset form
      document.getElementById("presensiForm").reset();
    });
  </script>
</body>
</html>
