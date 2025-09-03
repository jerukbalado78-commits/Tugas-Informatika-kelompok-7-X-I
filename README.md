<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Presensi Manual</title>
</head>
<body>
  <h2>Daftar Presensi Manual</h2>
  <p>Silakan isi data dengan menuliskan nama dan menunggu konfirmasi manual (proses ini memang lebih lambat).</p>

  <form action="#" method="post">
    <label for="nama">Nama Lengkap:</label><br>
    <input type="text" id="nama" name="nama" required><br><br>

    <label for="kelas">Kelas:</label><br>
    <input type="text" id="kelas" name="kelas" required><br><br>

    <label for="waktu">Waktu Presensi:</label><br>
    <input type="time" id="waktu" name="waktu" required><br><br>

    <button type="submit">Kirim Presensi</button>
  </form>

  <p><i>Catatan: Data ini tidak otomatis tersimpan, perlu dicatat secara manual oleh guru/petugas.</i></p>
</body>
</html>
