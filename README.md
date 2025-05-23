Berikut adalah README lengkap (siap untuk GitHub) untuk proyek kamu yang menghubungkan event TikTok (chat, like, gift, dsb) ke **IndoFinity** melalui **WebSocket/Socket.IO** dan menampilkan overlay ke **TikTok Live Studio**.

---

## 📦 TikTok Live Connector to IndoFinity Overlay

Proyek ini menghubungkan event TikTok LIVE (seperti gift, like, join, chat) menggunakan [`tiktok-live-connector`](https://www.npmjs.com/package/tiktok-live-connector) dan mengirimkan data secara **real-time** ke **IndoFinity** via Socket.IO (`http://localhost:62025`) untuk ditampilkan di TikTok Live Studio sebagai **custom overlay**.

---

## 📁 Struktur Folder

```
C:\Users\USER\tiktok-live-bot\
│
├── public\
│   └── overlay.html     # Halaman overlay (dipakai di TikTok Live Studio)
│
├── config.json          # Konfigurasi username TikTok
├── app.js               # Logic event TikTok
├── server.js            # Express + WebSocket server
└── package.json         # Konfigurasi dependencies Node.js
```

---

## ✅ Fitur

* Terhubung ke akun TikTok Live (real-time).
* Menerima event: `chat`, `like`, `gift`, `follow`, `share`, `join`.
* Kirim event ke **IndoFinity** via `Socket.IO`.
* Buat overlay HTML untuk ditampilkan di TikTok Live Studio.
* Custom list: Top Like, Gift, dll.

---

## 🛠️ Cara Instalasi (Windows)

### 1. Clone atau Download Proyek

```bash
cd C:\Users\USER\
git clone https://github.com/nama-anda/tiktok-live-bot.git
cd tiktok-live-bot
```

### 2. Buat File Konfigurasi

**config.json**

```json
{
  "username": "cilla1_popcard"
}
```

Ganti dengan username TikTok kamu yang sedang **LIVE**.

---

### 3. Install Dependency

```bash
npm init -y
npm install express socket.io socket.io-client tiktok-live-connector@1.5.8
```

> Pastikan kamu menggunakan versi `1.5.8` karena versi `2.x` belum tersedia di NPM.

---

### 4. Buat File `server.js`

```js
// server.js
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import path from 'path';
import { fileURLToPath } from 'url';

const app = express();
const server = createServer(app);
const io = new Server(server);
const PORT = 3000;

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Serve static file untuk overlay
app.use(express.static(path.join(__dirname, 'public')));

server.listen(PORT, () => {
  console.log(`✅ Server jalan di http://localhost:${PORT}`);
});

export default io;
```

---

### 5. Buat File `app.js`

```js
// app.js
import TikTokLiveConnection from 'tiktok-live-connector';
import fs from 'fs';
import io from './server.js';
import { io as clientIO } from 'socket.io-client';

// Load username dari config
const config = JSON.parse(fs.readFileSync('./config.json', 'utf8'));
const tiktokUsername = config.username;

// Connect ke TikTok
const tiktok = new TikTokLiveConnection(tiktokUsername);
const indofinitySocket = clientIO("http://localhost:62025");

// Forward semua event ke IndoFinity & klien
const forwardEvent = (event, data) => {
  console.log(`📡 Event: ${event}`, data);

  // Kirim ke client overlay
  io.emit(event, data);

  // Kirim ke IndoFinity
  indofinitySocket.emit("message", {
    event,
    data,
  });
};

// Daftar Event TikTok
tiktok.on('chat', (data) => forwardEvent('chat', data));
tiktok.on('gift', (data) => forwardEvent('gift', data));
tiktok.on('like', (data) => forwardEvent('like', data));
tiktok.on('follow', (data) => forwardEvent('follow', data));
tiktok.on('share', (data) => forwardEvent('share', data));
tiktok.on('join', (data) => forwardEvent('join', data));

// Handle koneksi
tiktok.connect().then(() => {
  console.log(`🔗 Terhubung ke TikTok: ${tiktokUsername}`);
}).catch((err) => {
  console.error("❌ Gagal konek ke TikTok:", err);
});
```

---

### 6. Buat File Overlay: `public/overlay.html`

```html
<!-- public/overlay.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Overlay TikTok</title>
  <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
  <style>
    body { margin: 0; font-family: sans-serif; background: transparent; color: white; }
    #likes { position: absolute; top: 10px; left: 10px; font-size: 20px; }
    .user { margin-top: 5px; }
  </style>
</head>
<body>
  <div id="likes">
    <strong>🔥 Top Likes:</strong>
    <div id="likeList"></div>
  </div>

  <script>
    const socket = io();
    const likeList = {};
    const likeContainer = document.getElementById('likeList');

    const updateLikes = () => {
      likeContainer.innerHTML = '';
      const sorted = Object.entries(likeList)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 5);
      for (const [user, count] of sorted) {
        const div = document.createElement('div');
        div.className = "user";
        div.textContent = `@${user} - ${count} tap`;
        likeContainer.appendChild(div);
      }
    };

    socket.on('like', (data) => {
      const user = data?.uniqueId || "unknown";
      likeList[user] = (likeList[user] || 0) + data.likeCount;
      updateLikes();
    });
  </script>
</body>
</html>
```

---

## ▶️ Jalankan Server

```bash
node app.js
```

---

## 📺 Cara Menampilkan di TikTok Live Studio

1. Buka TikTok Live Studio.
2. Tambah **Web Overlay**.
3. Masukkan URL:

```
http://localhost:3000/overlay.html
```

4. Klik "Tampilkan".

---

## 🧪 Testing

Gunakan username yang **sedang LIVE**, seperti:

```json
{
  "username": "cilla1_popcard"
}
```

Lalu lakukan **tap / gift** dari HP lain untuk melihat hasilnya muncul di layar overlay.

---

## ❓ FAQ

**Q: IndoFinity tidak menerima data?**
A: Pastikan `IndoFinity Socket` aktif di `http://localhost:62025`.

**Q: Overlay tidak muncul di TikTok Live Studio?**
A: Cek `localhost:3000/overlay.html` di browser. Pastikan file `public/overlay.html` ada dan tidak error.

---

## 🧼 Lisensi

MIT License. Bebas digunakan dan dimodifikasi.

---

Kalau kamu ingin saya upload contoh ini ke GitHub untuk kamu fork, tinggal bilang saja:
**"Upload ke GitHub"** – nanti saya buatkan public repo contoh.

Semoga README ini membantu!
