# <div align='center'>SakyKyzenn</div>

<div align='center'>

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub](https://img.shields.io/badge/source-GitHub-black.svg)](https://github.com/SakyXzy-offc/baileys)
[![Node](https://img.shields.io/badge/node-%3E%3D20-brightgreen.svg)](package.json)

</div>

<div align="center">
  <strong>SakyKyzenn</strong> — WhatsApp Multi-Device API untuk membangun bot dan aplikasi berbasis Node.js.
  <br>
  <sub>Developed and maintained by <strong>SakyXzy</strong>.</sub>
</div>

---

## 👨‍💻 Tentang Proyek

**Baileys** (versi SakyXzy) adalah fork dari [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys), dikembangkan dan dikelola oleh **SakyXzy**. README ini menjelaskan instalasi, konfigurasi, koneksi WhatsApp, dan penggunaan API.

---

## 🧾 Status Versi

| Komponen | Versi |
|---|---|
| WhatsApp (`WAProto.proto`) | `2.3000.1052536159` |
| Node.js minimum | `20.0.0` |
| Package | `@whiskeysockets/baileys@2.1.0` |

---

## ✨ Fitur Utama

- 🚀 **Modern & Cepat** – Dibangun dengan TypeScript dan Node.js 20+.
- 🔧 **Perbaikan @lid & @pn** – Mengatasi masalah ID lokal ke ID ponsel di grup WhatsApp.
- 📱 **Dukungan Multi-Device** – Koneksi penuh ke WhatsApp Multi-Device.
- 🔐 **Enkripsi End-to-End** – Semua pesan terenkripsi.
- 📨 **Semua Jenis Pesan** – Teks, media (gambar, video, audio), polling, stiker, lokasi, kontak, reaksi, dan lainnya.
- 🧩 **Ringan & Optimal** – Dukungan streaming media.
- 📂 **Manajemen Autentikasi** – Simpan kredensial di file atau database.

---

## ⚠️ Peringatan

Proyek ini **tidak berafiliasi, didukung, atau terhubung secara resmi** dengan WhatsApp atau anak perusahaannya. Situs resmi WhatsApp: [whatsapp.com](https://whatsapp.com).

Pengguna bertanggung jawab penuh atas penggunaan library ini. Hindari spam dan otomatisasi berlebihan yang melanggar Ketentuan Layanan WhatsApp.

---

## 📦 Instalasi

### Persyaratan

- Node.js **20 atau lebih baru** (dicek otomatis saat `preinstall`).

### Versi Stabil

```bash
npm install @whiskeysockets/baileys
```

### Versi Terbaru

```bash
npm install @whiskeysockets/baileys@latest
# atau dengan yarn
yarn add @whiskeysockets/baileys@latest
```

> Nama paket mengikuti `name` di `package.json`. Jika kamu mempublikasikan fork ini dengan nama sendiri, ganti nama di `package.json` dan ubah perintah instalasi di atas.

### Cara Impor

```javascript
// CommonJS
const { default: makeWASocket } = require('@whiskeysockets/baileys')

// ES Module
import makeWASocket from '@whiskeysockets/baileys'
```

---

## 🚀 Mulai Cepat

### Contoh Dasar (dengan QR Code)

```javascript
const { default: makeWASocket, DisconnectReason, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const { Boom } = require('@hapi/boom')

async function connectToWhatsApp() {
    // Simpan kredensial di folder 'auth_info'
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true,
        browser: ['SakyKyzenn', 'Desktop', '1.0.0']
    })

    // Event koneksi
    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        if (connection === 'close') {
            const shouldReconnect = (lastDisconnect?.error as Boom)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('Koneksi tertutup, reconnecting:', shouldReconnect)
            if (shouldReconnect) connectToWhatsApp()
        } else if (connection === 'open') {
            console.log('✅ Berhasil terhubung ke WhatsApp!')
        }
    })

    // Event pesan masuk
    sock.ev.on('messages.upsert', async ({ messages }) => {
        for (const m of messages) {
            if (!m.message) continue
            console.log('📱 Pesan baru:', JSON.stringify(m, undefined, 2))

            // Balas otomatis
            await sock.sendMessage(m.key.remoteJid, {
                text: 'Halo! Saya bot WhatsApp menggunakan SakyKyzenn 🤖'
            })
        }
    })

    // Simpan kredensial saat diperbarui
    sock.ev.on('creds.update', saveCreds)
}

connectToWhatsApp()
```

> Contoh di atas memakai JavaScript. Jika menggunakan TypeScript, tambahkan tipe sesuai kebutuhan (misalnya `m.key.remoteJid!`).

---

## 🔌 Koneksi Akun

WhatsApp menyediakan API multi-device yang memungkinkan Baileys terautentikasi sebagai klien WhatsApp sekunder melalui QR code atau pairing code.

### Hubungkan dengan QR Code

```javascript
const { default: makeWASocket, Browsers } = require('@whiskeysockets/baileys')

const sock = makeWASocket({
    browser: Browsers('SakyKyzenn'),
    printQRInTerminal: true
})
```

Setelah koneksi dimulai, QR code akan muncul di terminal. Pindai dengan WhatsApp di ponsel kamu untuk login.

> `printQRInTerminal` sudah tidak direkomendasikan oleh upstream Baileys. Jika kamu menangani QR sendiri, ambil string QR dari event `connection.update` (`qr`) dan render sesuai kebutuhan.

### Hubungkan dengan Pairing Code

Pairing code memungkinkan koneksi tanpa QR code. Nomor telepon harus tanpa `+`, `()`, atau `-`, dan wajib menyertakan kode negara.

```javascript
const { default: makeWASocket } = require('@whiskeysockets/baileys')

const sock = makeWASocket({
    printQRInTerminal: false
})

// Pairing normal
if (!sock.authState.creds.registered) {
    const number = '6285134816783' // ganti dengan nomor kamu
    const code = await sock.requestPairingCode(number)
    console.log('🔑 Kode Pairing:', code)
}

// Pairing dengan kode kustom (8 karakter)
if (!sock.authState.creds.registered) {
    const pair = 'SAKYKYZE'
    const number = '6285134816783'
    const code = await sock.requestPairingCode(number, pair)
    console.log('🔑 Kode Pairing Kustom:', code)
}
```

### Terima Riwayat Lengkap

Untuk menerima riwayat pesan lengkap, atur `syncFullHistory` ke `true`. Nilai default-nya sudah `true` di `lib/Defaults`, jadi hanya perlu ditulis jika ingin eksplisit.

```javascript
const { default: makeWASocket, Browsers } = require('@whiskeysockets/baileys')

const sock = makeWASocket({
    browser: Browsers('Desktop'),
    syncFullHistory: true
})
```

---

## ⚙️ Konfigurasi Socket

### Caching Metadata Grup (Direkomendasikan)

Untuk penggunaan grup, implementasikan caching metadata grup. Contoh ini memakai paket `node-cache` (`npm install node-cache`), yang bukan dependensi bawaan library.

```javascript
const { default: makeWASocket } = require('@whiskeysockets/baileys')
const NodeCache = require('node-cache')

const groupCache = new NodeCache({ stdTTL: 5 * 60, useClones: false })

const sock = makeWASocket({
    cachedGroupMetadata: async (jid) => groupCache.get(jid)
})

sock.ev.on('groups.update', async ([event]) => {
    const metadata = await sock.groupMetadata(event.id)
    groupCache.set(event.id, metadata)
})

sock.ev.on('group-participants.update', async (event) => {
    const metadata = await sock.groupMetadata(event.id)
    groupCache.set(event.id, metadata)
})
```

### Perbaiki Sistem Retry & Dekripsi Vote Poll

Tingkatkan pengiriman ulang pesan dan dekripsi vote poll dengan store pesan:

```javascript
const sock = makeWASocket({
    getMessage: async (key) => await getMessageFromStore(key) // implementasikan sendiri
})
```

### Terima Notifikasi di Aplikasi WhatsApp

Nonaktifkan status online agar notifikasi tetap masuk ke ponsel:

```javascript
const sock = makeWASocket({
    markOnlineOnConnect: false
})
```

---

## 💾 Simpan Info Auth

Hindari pemindaian QR code berulang dengan menyimpan kredensial:

```javascript
const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')

async function connect() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')
    const sock = makeWASocket({ auth: state })
    sock.ev.on('creds.update', saveCreds)
}

connect()
```

> `useMultiFileAuthState` menyimpan status auth ke folder. Untuk produksi, gunakan database SQL/NoSQL dan kelola pembaruan kunci dengan hati-hati.

---

## 🗄️ Implementasi Data Store

Library tidak menyertakan penyimpanan default untuk obrolan, kontak, atau pesan. Namun, tersedia implementasi in-memory sederhana.

```javascript
const { default: makeWASocket, makeInMemoryStore } = require('@whiskeysockets/baileys')

const store = makeInMemoryStore({ })

store.readFromFile('./sakykyzenn_store.json')

setInterval(() => {
    store.writeToFile('./sakykyzenn_store.json')
}, 10_000)

const sock = makeWASocket({ })
store.bind(sock.ev)

sock.ev.on('chats.upsert', () => {
    console.log('dapat obrolan', store.chats.all())
})

sock.ev.on('contacts.upsert', () => {
    console.log('dapat kontak', Object.values(store.contacts))
})
```

---

## 🆔 Penjelasan ID WhatsApp

`jid` adalah ID WhatsApp untuk orang atau grup tujuan pesan.

- Orang: `[kode negara][nomor telepon]@s.whatsapp.net`
  - Contoh: `19999999999@s.whatsapp.net`
- Grup: `123456789-123345@g.us`
- Daftar siaran: `[timestamp pembuatan]@broadcast`
- Status/cerita: `status@broadcast`

---

## 🔧 Fungsi Utilitas

- `getContentType` – Mengembalikan tipe konten pesan
- `getDevice` – Mengembalikan perangkat pengirim pesan
- `makeCacheableSignalKeyStore` – Mempercepat store auth
- `downloadMediaMessage` – Unduh konten dari pesan media

---

## 📤 Kirim Pesan

Semua jenis pesan dikirim dengan satu fungsi:

```javascript
sock.sendMessage(jid, content, options)
```

### Pesan Non-Media

**Pesan Teks**

```javascript
await sock.sendMessage(jid, { text: 'halo dunia' })
```

**Pesan Kutipan (bekerja dengan semua tipe)**

```javascript
await sock.sendMessage(jid, { text: 'halo dunia' }, { quoted: message })
```

**Sebut Pengguna**

```javascript
await sock.sendMessage(
    jid,
    {
        text: '@12345678901',
        mentions: ['12345678901@s.whatsapp.net']
    }
)
```

**Teruskan Pesan**

```javascript
const msg = getMessageFromStore() // implementasikan sendiri
await sock.sendMessage(jid, { forward: msg })
```

**Pesan Lokasi**

```javascript
await sock.sendMessage(
    jid,
    {
        location: {
            degreesLatitude: 24.121231,
            degreesLongitude: 55.1121221
        }
    }
)
```

**Pesan Kontak (vCard)**

```javascript
const vcard = 'BEGIN:VCARD\n'
            + 'VERSION:3.0\n'
            + 'FN:Jeff Singh\n'
            + 'ORG:Ashoka Uni;\n'
            + 'TEL;type=CELL;type=VOICE;waid=911234567890:+91 12345 67890\n'
            + 'END:VCARD'

await sock.sendMessage(
    jid,
    {
        contacts: {
            displayName: 'Jeff',
            contacts: [{ vcard }]
        }
    }
)
```

**Pesan Reaksi**

```javascript
await sock.sendMessage(
    jid,
    {
        react: {
            text: '💖', // kosongkan untuk hapus reaksi
            key: message.key
        }
    }
)
```

**Pin Pesan**

```javascript
await sock.sendMessage(
    jid,
    {
        pin: {
            type: 1, // 0 untuk hapus
            time: 86400, // 24 jam dalam detik
            key: message.key
        }
    }
)
```

**Pesan Poll**

```javascript
await sock.sendMessage(
    jid,
    {
        poll: {
            name: 'Poll Saya',
            values: ['Opsi 1', 'Opsi 2', 'Opsi 3'],
            selectableCount: 1,
            toAnnouncementGroup: false
        }
    }
)
```

**Kirim dengan Pratinjau Tautan**

Instal `link-preview-js` terlebih dahulu:

```bash
npm install link-preview-js
```

```javascript
await sock.sendMessage(
    jid,
    {
        text: 'Halo, ini dikirim menggunakan https://npmjs.com/@whiskeysockets/baileys'
    }
)
```

### Pesan Media

Di pesan media, sumber file bisa berupa `{ url: string }`, `{ stream: Readable }`, atau `Buffer` langsung.

**Kirim Gambar**

```javascript
await sock.sendMessage(
    jid,
    {
        image: {
            url: './Media/ma_img.png'
        },
        caption: 'halo dunia'
    }
)
```

**Kirim Video**

```javascript
await sock.sendMessage(
    jid,
    {
        video: {
            url: './Media/ma_video.mp4'
        },
        caption: 'halo dunia',
        ptv: false // true untuk video note
    }
)
```

**Kirim GIF**

```javascript
const fs = require('fs')

await sock.sendMessage(
    jid,
    {
        video: fs.readFileSync('Media/ma_gif.mp4'),
        caption: 'halo dunia',
        gifPlayback: true
    }
)
```

**Kirim Audio**

```javascript
await sock.sendMessage(
    jid,
    {
        audio: {
            url: './Media/audio.mp3'
        },
        mimetype: 'audio/mp4'
    }
)
```

**View Once**

```javascript
await sock.sendMessage(
    jid,
    {
        image: {
            url: './Media/ma_img.png'
        },
        viewOnce: true,
        caption: 'halo dunia'
    }
)
```

---

## ✏️ Ubah Pesan

**Hapus Pesan (untuk semua)**

```javascript
const msg = await sock.sendMessage(jid, { text: 'halo dunia' })
await sock.sendMessage(jid, { delete: msg.key })
```

**Edit Pesan**

```javascript
const response = await sock.sendMessage(jid, { text: 'teks awal' })
await sock.sendMessage(jid, {
    text: 'teks terupdate di sini',
    edit: response.key,
})
```

---

## 🖼️ Manipulasi Pesan Media

**Unduh Pesan Media**

```javascript
const { createWriteStream } = require('fs')
const { downloadMediaMessage, getContentType } = require('@whiskeysockets/baileys')

sock.ev.on('messages.upsert', async ({ messages: [m] }) => {
    if (!m.message) return
    const messageType = getContentType(m.message)

    if (messageType === 'imageMessage') {
        const stream = await downloadMediaMessage(
            m,
            'stream',
            { },
            {
                logger,
                reuploadRequest: sock.updateMediaMessage
            }
        )
        const writeStream = createWriteStream('./my-download.jpeg')
        stream.pipe(writeStream)
    }
})
```

> `logger` adalah instance logger yang kamu siapkan sendiri (misalnya dari `pino`).

---

## 📞 Tolak Panggilan

```javascript
await sock.rejectCall(callId, callFrom)
```

---

## 💬 Status & Kehadiran di Obrolan

**Baca Pesan**

```javascript
await sock.readMessages([key]) // bisa beberapa kunci sekaligus
```

**Update Kehadiran**

```javascript
await sock.sendPresenceUpdate('available', jid)
await sock.sendPresenceUpdate('composing', jid)
```

---

## 📁 Ubah Obrolan

**Arsipkan Obrolan**

```javascript
const lastMsgInChat = await getLastMessageInChat(jid) // implementasikan sendiri
await sock.chatModify({ archive: true, lastMessages: [lastMsgInChat] }, jid)
```

**Bisukan/Buka Bisukan Obrolan**

```javascript
// Bisukan 8 jam
await sock.chatModify({ mute: 8 * 60 * 60 * 1000 }, jid)
// Buka bisukan
await sock.chatModify({ mute: null }, jid)
```

**Tandai Obrolan Dibaca/Belum Dibaca**

```javascript
const lastMsgInChat = await getLastMessageInChat(jid) // implementasikan sendiri
await sock.chatModify({ markRead: false, lastMessages: [lastMsgInChat] }, jid)
```

**Hapus Pesan untuk Saya**

```javascript
await sock.chatModify(
    {
        clear: {
            messages: [
                {
                    id: 'ATWYHDNNWU81732J',
                    fromMe: true,
                    timestamp: '1654823909'
                }
            ]
        }
    },
    jid
)
```

**Hapus Obrolan**

```javascript
const lastMsgInChat = await getLastMessageInChat(jid) // implementasikan sendiri
await sock.chatModify({
    delete: true,
    lastMessages: [
        {
            key: lastMsgInChat.key,
            messageTimestamp: lastMsgInChat.messageTimestamp
        }
    ]
}, jid)
```

**Pin/Lepas Pin Obrolan**

```javascript
await sock.chatModify({ pin: true }, jid)  // pin
await sock.chatModify({ pin: false }, jid) // lepas pin
```

**Beri Bintang/Hapus Bintang Pesan**

```javascript
await sock.chatModify({
    star: {
        messages: [
            {
                id: 'messageID',
                fromMe: true
            }
        ],
        star: true // false untuk hapus bintang
    }
}, jid)
```

**Pesan Menghilang**

```javascript
// Nyalakan pesan menghilang (7 hari)
await sock.sendMessage(
    jid,
    { disappearingMessagesInChat: 604800 } // 7 hari dalam detik
)

// Kirim sebagai pesan menghilang
await sock.sendMessage(jid, { text: 'halo' }, { ephemeralExpiration: 604800 })

// Matikan
await sock.sendMessage(
    jid,
    { disappearingMessagesInChat: false }
)
```

---

## 👤 Query Pengguna

**Periksa ID Ada di WhatsApp**

```javascript
const [result] = await sock.onWhatsApp(jid)
if (result.exists) console.log(`${jid} ada di WhatsApp, sebagai jid: ${result.jid}`)
```

**Query Riwayat Obrolan**

```javascript
const msg = await getOldestMessageInChat(jid) // implementasikan sendiri
await sock.fetchMessageHistory(
    50, // jumlah (maks 50 per query)
    msg.key,
    msg.messageTimestamp
)
```

**Ambil Status**

```javascript
const status = await sock.fetchStatus(jid)
console.log('status: ' + status)
```

**Ambil Foto Profil (grup juga)**

```javascript
// resolusi rendah
const ppUrl = await sock.profilePictureUrl(jid)
console.log(ppUrl)

// resolusi tinggi
const ppUrlHigh = await sock.profilePictureUrl(jid, 'image')
```

**Ambil Profil Bisnis**

```javascript
const profile = await sock.getBusinessProfile(jid)
console.log('deskripsi: ' + profile.description + ', kategori: ' + profile.category)
```

**Pantau Kehadiran Seseorang**

```javascript
sock.ev.on('presence.update', console.log)
await sock.presenceSubscribe(jid)
```

---

## 🔄 Ubah Profil

**Ubah Status Profil**

```javascript
await sock.updateProfileStatus('Halo Dunia!')
```

**Ubah Nama Profil**

```javascript
await sock.updateProfileName('Nama Saya')
```

**Ubah Foto Profil (grup juga)**

```javascript
await sock.updateProfilePicture(jid, { url: './new-profile-picture.jpeg' })
```

**Hapus Foto Profil (grup juga)**

```javascript
await sock.removeProfilePicture(jid)
```

---

## 👥 Grup

**Buat Grup**

```javascript
const group = await sock.groupCreate('Grup Saya', ['1234@s.whatsapp.net', '4564@s.whatsapp.net'])
console.log('grup dibuat dengan id: ' + group.gid)
```

**Tambah/Hapus atau Turunkan/Naikkan Admin**

```javascript
await sock.groupParticipantsUpdate(
    jid,
    ['abcd@s.whatsapp.net', 'efgh@s.whatsapp.net'],
    'add' // 'remove', 'demote', atau 'promote'
)
```

**Ubah Subjek (nama)**

```javascript
await sock.groupUpdateSubject(jid, 'Subjek Baru!')
```

**Ubah Deskripsi**

```javascript
await sock.groupUpdateDescription(jid, 'Deskripsi Baru!')
```

**Ubah Pengaturan**

```javascript
// Hanya admin yang bisa kirim pesan
await sock.groupSettingUpdate(jid, 'announcement')
// Semua anggota bisa kirim pesan
await sock.groupSettingUpdate(jid, 'not_announcement')
// Semua anggota bisa ubah pengaturan grup
await sock.groupSettingUpdate(jid, 'unlocked')
// Hanya admin bisa ubah pengaturan grup
await sock.groupSettingUpdate(jid, 'locked')
```

**Keluar Grup**

```javascript
await sock.groupLeave(jid)
```

**Ambil Kode Undangan**

```javascript
const code = await sock.groupInviteCode(jid)
console.log('https://chat.whatsapp.com/' + code)
```

**Cabut Kode Undangan**

```javascript
const code = await sock.groupRevokeInvite(jid)
console.log('Kode grup baru: ' + code)
```

**Gabung Menggunakan Kode Undangan**

```javascript
const response = await sock.groupAcceptInvite(code)
console.log('bergabung ke: ' + response)
```

**Ambil Info Grup dari Kode Undangan**

```javascript
const response = await sock.groupGetInviteInfo(code)
console.log('info grup: ', response)
```

**Query Metadata**

```javascript
const metadata = await sock.groupMetadata(jid)
console.log(metadata.id + ', judul: ' + metadata.subject + ', deskripsi: ' + metadata.desc)
```

**Ambil Semua Metadata Grup yang Diikuti**

```javascript
const response = await sock.groupFetchAllParticipating()
console.log(response)
```

**Toggle Pesan Sementara (Ephemeral)**

```javascript
await sock.groupToggleEphemeral(jid, 86400) // 24 jam
```

**Ubah Mode Tambah Anggota**

```javascript
await sock.groupMemberAddMode(
    jid,
    'all_member_add' // atau 'admin_add'
)
```

---

## 🔒 Privasi

**Blokir/Buka Blokir Pengguna**

```javascript
await sock.updateBlockStatus(jid, 'block')
await sock.updateBlockStatus(jid, 'unblock')
```

**Ambil Pengaturan Privasi**

```javascript
const privacySettings = await sock.fetchPrivacySettings(true)
console.log(privacySettings)
```

**Ambil Daftar Blokir**

```javascript
const blocklist = await sock.fetchBlocklist()
console.log(blocklist)
```

**Update Privasi**

```javascript
await sock.updateLastSeenPrivacy('contacts')
await sock.updateOnlinePrivacy('all')
await sock.updateProfilePicturePrivacy('contacts')
await sock.updateStatusPrivacy('none')
await sock.updateReadReceiptsPrivacy('all')
await sock.updateGroupsAddPrivacy('contacts')
await sock.updateDefaultDisappearingMode(86400) // 24 jam
```

---

## 📢 Status & Siaran

**Kirim Status/Cerita**

```javascript
await sock.sendMessage(
    'status@broadcast',
    {
        image: {
            url: url
        },
        caption: caption
    },
    {
        backgroundColor: backgroundColor,
        font: font,
        statusJidList: statusJidList,
        broadcast: true
    }
)
```

---

## 🛠️ Fungsi Kustom

**Aktifkan Level Debug**

```javascript
const pino = require('pino')
const { default: makeWASocket } = require('@whiskeysockets/baileys')

const sock = makeWASocket({
    logger: pino({ level: 'debug' }),
})
```

**Daftarkan Callback untuk Event WebSocket**

```javascript
// Untuk tag 'edge_routing'
sock.ws.on('CB:edge_routing', (node) => { })

// Dengan id 'abcd'
sock.ws.on('CB:edge_routing,id:abcd', (node) => { })

// Dengan node routing_info
sock.ws.on('CB:edge_routing,id:abcd,routing_info', (node) => { })
```

---

## 💡 Tips & Praktik Terbaik

- **Gunakan Caching** – Cache metadata grup untuk performa lebih baik.
- **Tangani Rekoneksi** – Implementasikan auto-reconnect untuk kestabilan.
- **Kelola Store** – Gunakan database (MongoDB, PostgreSQL) untuk produksi.
- **Penanganan Error** – Bungkus panggilan socket dengan try-catch.
- **Batas Rate** – Hormati batas WhatsApp untuk menghindari ban.
- **Keamanan** – Jangan bagikan kredensial auth; gunakan variabel lingkungan.

---

## 🔄 Pembaruan Terbaru

- **WAProto** diperbarui ke `2.3000.1052536159`. Termasuk message dan enum baru untuk fitur AI, bisnis, backup, wallpaper, panggilan, dan enkripsi multi-device.
- Message `BotAvatarMetadata` dihapus dari proto.
- Field `aIRichResponseContentItem` diganti `aiRichResponseContentItem`.
- `fflate` ditambahkan sebagai dependensi.

Untuk generate ulang file proto:

```bash
yarn gen:protobuf
```

---

## 📄 Lisensi

Didistribusikan di bawah **Lisensi MIT**. Kode ini merupakan fork dari WhiskeySockets/Baileys yang juga berlisensi MIT; notice copyright upstream dipertahankan di file [LICENSE](LICENSE).

---

<div align="center">
  <strong>SakyKyzenn</strong> — by SakyXzy<br>
  WhatsApp Multi-Device API dengan peningkatan kompatibilitas @lid dan @pn.
  <br><br>
  <sub>Maintained by <a href="https://github.com/SakyXzy-offc">SakyXzy</a></sub>
</div>
