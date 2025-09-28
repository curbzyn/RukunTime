<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>RukunTime</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h1 { color: #006666; }
    #times { margin-top: 20px; }
    .time-item { margin: 5px 0; font-size: 18px; }
    button { margin-top: 20px; padding: 10px 15px; border: none; background: #009688; color: white; cursor: pointer; border-radius: 8px; }
    button:hover { background: #00796b; }
  </style>        
</head>
<body>
  <h1>Jadwal Sholat Harian</h1>
  <p id="status">Mengambil lokasi...</p>
  <div id="times"></div>
  <button id="findMosque">Cari Masjid Terdekat</button>

  <!-- PrayTimes library -->
  <script src="https://cdn.jsdelivr.net/npm/praytimes@2.3.2/PrayTimes.js"></script>
  <script src="script.js"></script>
</body>
</html>
const statusEl = document.getElementById("status");
const timesEl = document.getElementById("times");
const btnMosque = document.getElementById("findMosque");

let userLat, userLon;

// Minta izin notifikasi
if ("Notification" in window && Notification.permission !== "granted") {
  Notification.requestPermission();
}

// Fungsi menampilkan jadwal sholat
function displayTimes(times) {
  timesEl.innerHTML = "";
  for (let [name, time] of Object.entries(times)) {
    const div = document.createElement("div");
    div.className = "time-item";
    div.textContent = `${name}: ${time}`;
    timesEl.appendChild(div);

    // Jadwalkan notifikasi jika waktu belum lewat
    scheduleNotification(name, time);
  }
}

// Jadwalkan notifikasi sholat
function scheduleNotification(name, time) {
  if (!("Notification" in window) || Notification.permission !== "granted") return;

  const now = new Date();
  const [h, m] = time.split(":").map(Number);
  const prayerTime = new Date(now);
  prayerTime.setHours(h, m, 0, 0);

  if (prayerTime > now) {
    const timeout = prayerTime.getTime() - now.getTime();
    setTimeout(() => {
      new Notification(`Waktu Sholat ${name}`, {
        body: `Saatnya menunaikan sholat ${name}.`,
        icon: "https://cdn-icons-png.flaticon.com/512/3500/3500833.png"
      });
    }, timeout);
  }
}

// Ambil lokasi
if (navigator.geolocation) {parseFloat('stringNumber');
  navigator.geolocation.getCurrentPosition(pos => {
    userLat = pos.coords.latitude;
    userLon = pos.coords.longitude;
    statusEl.textContent = `Lokasi: ${userLat.toFixed(4)}, ${userLon.toFixed(4)}`;

    // Atur metode perhitungan (MWL default)
    const prayTimes = new PrayTimes("MWL");

    // Ambil timezone offset
    const now = new Date();
    const tz = -now.getTimezoneOffset() / 60;

    // Hitung jadwal sholat
    const times = prayTimes.getTimes(now, [userLat, userLon], tz);
    displayTimes(times);

    // Tombol cari masjid
    btnMosque.onclick = () => {
      const url = `https://www.google.com/maps/search/mosque/@${userLat},${userLon},15z`;
      window.open(url, "_blank");
    };

  }, err => {
    statusEl.textContent = "Gagal mendapatkan lokasi: " + err.message;
  });
} else {
  statusEl.textContent = "Browser tidak mendukung geolocation.";
}
