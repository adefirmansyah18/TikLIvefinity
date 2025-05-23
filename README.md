Berikut adalah file `README.md` **lengkap dan rapi** untuk kamu simpan di GitHub. Isinya sudah disesuaikan agar kamu, sebagai pemula, bisa membaca ulang dengan mudah dan menjalankan ulang proyek kapan saja.

---

## 📺 TikTok Live Bot + IndoFinity Overlay

**Menampilkan event TikTok (like, chat, gift, dll) ke layar TikTok Live Studio melalui overlay dan mengirimkan ke IndoFinity WebSocket (Socket.IO)**
Username Live: `cilla1_popcard`

---

### 📁 Struktur Folder

```
tiktok-live-bot/
│
├── server.js               <- Server utama: koneksi TikTok & IndoFinity
├── config.json             <- Konfigurasi username TikTok
├── package.json            <- Info dependencies
├── public/
│   └── overlay.html        <- File overlay untuk ditampilkan di TikTok Studio
```

---

### ✅ 1. Setup Awal

#### Langkah pertama:

Install Node.js v18+ dari [https://nodejs.org/](https://nodejs.org/)

#### Jalankan terminal:

```bash
cd C:\Users\USER\tiktok-live-bot
npm install
```

---

### ✅ 2. Isi File-File Proyek

#### `config.json`

```json
{
  "username": "cilla1_popcard"
}
```

---

#### `package.json`

```json
{
  "name": "tiktok-live-bot",
  "version": "1.0.0",
  "type": "module",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "socket.io": "^4.7.5",
    "socket.io-client": "^4.7.5",
    "tiktok-live-connector": "^2.0.5"
  }
}
```

---

#### `server.js`

```js
import express from 'express';
import http from 'http';
import { Server } from 'socket.io';
import { io as ClientIO } from 'socket.io-client';
import { WebcastPushConnection } from 'tiktok-live-connector';
import path from 'path';
import { fileURLToPath } from 'url';
import fs from 'fs';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Config
const config = JSON.parse(fs.readFileSync('./config.json'));
const tiktokUsername = config.username;

// Express & Socket.IO Server
const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static(path.join(__dirname, 'public')));
server.listen(3000, () => {
  console.log('✅ Server running at http://localhost:3000');
});

// IndoFinity WebSocket Connection (Socket.IO)
const indofinitySocket = ClientIO('http://localhost:62025');

indofinitySocket.on('connect', () => {
  console.log('🛰️ Terhubung ke IndoFinity');
});
indofinitySocket.on('connect_error', (err) => {
  console.error('❌ Gagal konek ke IndoFinity:', err);
});

// TikTok LIVE Connection
const tiktok = new WebcastPushConnection(tiktokUsername);
tiktok.connect().then(() => {
  console.log(`📡 Terhubung ke TikTok LIVE: ${tiktokUsername}`);
}).catch(err => {
  console.error('❌ TikTok Connect Error:', err);
});

// Event TikTok
tiktok.on('chat', data => {
  const payload = { event: 'chat', data };
  console.log(`[CHAT] ${data.uniqueId}: ${data.comment}`);
  io.emit('message', payload);
  indofinitySocket.emit('message', payload);
});

tiktok.on('like', data => {
  const payload = { event: 'like', data };
  console.log(`[LIKE] ${data.uniqueId} memberi like`);
  io.emit('message', payload);
  indofinitySocket.emit('message', payload);
});
```

---

#### `public/overlay.html`

```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>Overlay TikTok</title>
  <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
  <style>
    body {
      margin: 0;
      background: transparent;
      font-family: Arial, sans-serif;
      color: white;
      overflow: hidden;
    }
    #event-log {
      position: absolute;
      top: 10px;
      left: 10px;
      width: 90%;
    }
    .event {
      margin-bottom: 10px;
      background: rgba(0, 0, 0, 0.5);
      padding: 8px 12px;
      border-radius: 12px;
      animation: fade 5s ease-out forwards;
    }
    @keyframes fade {
      0% { opacity: 1; transform: translateY(0); }
      90% { opacity: 1; }
      100% { opacity: 0; transform: translateY(-20px); }
    }
  </style>
</head>
<body>
  <div id="event-log"></div>

  <script>
    const socket = io();

    socket.on('connect', () => {
      console.log('✅ Overlay terhubung');
    });

    socket.on('message', ({ event, data }) => {
      const log = document.getElementById('event-log');
      const el = document.createElement('div');
      el.className = 'event';

      if (event === 'like') {
        el.innerText = `❤️ ${data.uniqueId} memberi like (${data.likeCount})`;
      } else if (event === 'chat') {
        el.innerText = `💬 ${data.uniqueId}: ${data.comment}`;
      } else {
        el.innerText = `[${event}] ${JSON.stringify(data)}`;
      }

      log.appendChild(el);
      setTimeout(() => el.remove(), 5000);
    });
  </script>
</body>
</html>
```

---

### ▶️ Cara Menjalankan

1. Buka terminal:

```bash
cd C:\Users\USER\tiktok-live-bot
npm install
node server.js
```

2. Buka browser:

```
http://localhost:3000/public/overlay.html
```

3. Masukkan link itu ke TikTok Live Studio → **Browser Source**
   Gunakan ukuran: **1280x720** atau sesuai layar live kamu.

---

### 🚀 Fitur Saat Ini

* Menerima event **chat** dan **like**
* Kirim ke **IndoFinity (Socket.IO port 62025)**
* Tampilkan ke layar lewat **overlay**
* Bisa dikembangkan untuk **gift**, **join**, **ranking**, dan lainnya

---

### 🛠️ Next Upgrade?

* 🔝 Leaderboard Top Like
* 🎁 Animasi Gift (Top Gifter)
* 🎉 Tampilan Join / Follow
* 🎨 Kustomisasi tema & efek

---

### 🤝 Credits

* [tiktok-live-connector](https://github.com/zerodytrash/TikTok-Live-Connector)
* IndoFinity for overlay Socket.IO endpoint

---

Jika kamu ingin saya bantu kembangkan lebih lanjut, cukup bilang:
**"Lanjutkan ke Top Like / Gift / Animasi masuk"**
Saya siap bantu 💡

---

Kamu bisa **copy-paste** isi ini ke file `README.md` di GitHub repo kamu, lalu tinggal commit.

Perlu bantuan commit ke GitHub juga? Kasih tahu saja!
