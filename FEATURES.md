# 🚀 Baileys Fork - Exclusive Features Documentation
> **Fork oleh:** Tuan Pai ([@paikynie](https://github.com/paikynie))  
> **Repository:** [paikynie/Baileys](https://github.com/paikynie/Baileys)

Dokumentasi lengkap untuk semua fitur eksklusif yang telah ditambahkan ke dalam fork Baileys ini.

---

## 📋 Daftar Fitur

| No | Fitur | File Terdampak | Status |
|----|-------|----------------|--------|
| 1 | [AIRich (Meta AI Interactive UI)](#1-airich-meta-ai-interactive-ui) | `Types/Message.ts`, `Utils/messages.ts` | ✅ Stabil |
| 2 | [Advanced Status (Close Friends, Group, Custom BG)](#2-advanced-status) | `Types/Message.ts`, `Utils/messages.ts` | ✅ Stabil |
| 3 | [proxyMetaAI (Hijack Meta AI Resmi)](#3-proxymetaai) | `Socket/messages-send.ts` | ✅ Stabil |

---

## 1. AIRich (Meta AI Interactive UI)

### Apa itu AIRich?
WhatsApp mulai memblokir `InteractiveMessage` (pesan dengan tombol-tombol cantik) untuk akun non-bisnis resmi. Pesan tersebut menampilkan popup **"Message not supported, upgrade WhatsApp"** di HP penerima.

Fork ini **secara otomatis mem-bypass pembatasan tersebut** dengan cara:
- Membungkus pesan interaktif di dalam `viewOnceMessage` (FutureProofMessage)
- Menyuntikkan `botMetadata` agar WhatsApp mengenali bot ini sebagai AI resmi
- Tampilan akhir di HP penerima: **balon pesan abu-abu elegan khas Meta AI** ✨

### Tipe Data (`InteractiveMessageContent`)

```typescript
type InteractiveButton = {
    name: string          // Tipe tombol (lihat daftar di bawah)
    buttonParamsJson: string // Parameter tombol dalam format JSON string
}

type InteractiveMessageContent = {
    rich: {
        header?: {
            title?: string
            subtitle?: string
            hasMediaAttachment?: boolean
        }
        body?: {
            text: string
        }
        footer?: {
            text: string
        }
        buttons?: InteractiveButton[]
    }
}
```

### Tipe-tipe Tombol (Buttons) yang Didukung

| Nama | Fungsi | Contoh Parameter |
|------|--------|------------------|
| `quick_reply` | Tombol balas cepat | `{ display_text: "Halo", id: "my_id" }` |
| `cta_url` | Tombol buka URL | `{ display_text: "Kunjungi", url: "https://..." }` |
| `cta_copy` | Tombol salin teks | `{ display_text: "Salin Kode", copy_code: "DISKON50" }` |
| `cta_call` | Tombol telepon | `{ display_text: "Hubungi Kami", phone_number: "+628xxx" }` |
| `send_location` | Tombol kirim lokasi | `{ display_text: "Kirim Lokasi" }` |
| `address_message` | Tombol alamat | `{ display_text: "Masukkan Alamat" }` |

### Contoh Kode Lengkap

#### Contoh 1: Pesan dengan Tombol Sederhana
```typescript
await sock.sendMessage(jid, {
    rich: {
        header: {
            title: "🤖 Bot Menu"
        },
        body: {
            text: "Selamat datang! Silakan pilih menu:"
        },
        footer: {
            text: "Bot by Tuan Pai"
        },
        buttons: [
            {
                name: "quick_reply",
                buttonParamsJson: JSON.stringify({
                    display_text: "📋 Menu Utama",
                    id: "menu_utama"
                })
            },
            {
                name: "quick_reply",
                buttonParamsJson: JSON.stringify({
                    display_text: "ℹ️ Info Bot",
                    id: "info_bot"
                })
            }
        ]
    }
})
```

#### Contoh 2: Tombol Buka URL + Tombol Salin Kode
```typescript
await sock.sendMessage(jid, {
    rich: {
        body: {
            text: "Gunakan kode promo ini untuk diskon 50%!"
        },
        footer: {
            text: "Berlaku hingga 31 Agustus 2026"
        },
        buttons: [
            {
                name: "cta_copy",
                buttonParamsJson: JSON.stringify({
                    display_text: "📋 Salin Kode Promo",
                    copy_code: "TUANPAI50"
                })
            },
            {
                name: "cta_url",
                buttonParamsJson: JSON.stringify({
                    display_text: "🛒 Belanja Sekarang",
                    url: "https://contoh-toko.com"
                })
            }
        ]
    }
})
```

#### Cara Menangkap Klik Tombol
```typescript
sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0]
    
    // Tangkap interaksi dari tombol quick_reply
    const response = msg.message?.interactiveResponseMessage
    if (response) {
        const buttonId = JSON.parse(response.nativeFlowResponseMessage?.paramsJson || '{}').id
        
        if (buttonId === 'menu_utama') {
            await sock.sendMessage(msg.key.remoteJid!, { text: "Kamu memilih Menu Utama!" })
        }
    }
})
```

---

## 2. Advanced Status

### Apa itu Advanced Status?
Fitur bawaan Baileys tidak menyediakan cara mudah untuk:
- Mengirim status dengan **warna latar belakang custom**
- Mengirim status hanya untuk **Close Friends** (lingkaran hijau di WA)
- Mengirim status yang muncul sebagai **Group Status** (lingkaran biru tua)

Fork ini menambahkan tipe baru `StatusMessageContent` yang langsung bisa digunakan!

### Tipe Data (`StatusMessageContent`)

```typescript
type StatusMessageContent = {
    status: {
        text?: string           // Teks status (untuk status teks)
        image?: WAMediaUpload   // Gambar status
        video?: WAMediaUpload   // Video status
        color?: string          // Warna background ARGB (format: '#FFRRGGBB')
        font?: number           // Jenis font: 1-5
        audience?: 'all' | 'close_friends' | 'group'  // Target audiens
    }
} & Contextable
```

### Format Warna (`color`)

Format warna menggunakan **ARGB** (Alpha + RGB dalam Hex):

| Warna | Kode ARGB |
|-------|-----------|
| Hitam | `#FF000000` |
| Putih | `#FFFFFFFF` |
| Merah | `#FFFF0000` |
| Hijau Tua | `#FF008000` |
| Biru | `#FF0000FF` |
| Hijau WA | `#FF25D366` |
| Ungu | `#FF800080` |

### Contoh Kode Lengkap

#### Contoh 1: Status Teks Biasa dengan Warna
```typescript
await sock.sendMessage('status@broadcast', {
    status: {
        text: "Hai semua! Ini status dengan warna keren dari bot! 🎨",
        color: '#FF25D366', // Warna hijau WhatsApp
        font: 2
    }
})
```

#### Contoh 2: Status Close Friends (Lingkaran Hijau)
```typescript
// Siapkan daftar JID Close Friends kamu
const closeFriendsJids = [
    '628111111111@s.whatsapp.net',
    '628222222222@s.whatsapp.net'
]

await sock.sendMessage('status@broadcast', {
    status: {
        text: "Ini status rahasia, hanya Close Friends yang bisa lihat! 🤫",
        color: '#FF000000', // Background hitam
        font: 3,
        audience: 'close_friends' // WAJIB untuk Close Friends
    }
}, {
    statusJidList: closeFriendsJids // WAJIB: list penerima
})
```

#### Contoh 3: Group Status (Lingkaran Biru Tua)
```typescript
// Ambil daftar anggota grup terlebih dahulu
const groupMetadata = await sock.groupMetadata('1234567890@g.us')
const participantJids = groupMetadata.participants.map(p => p.id)

await sock.sendMessage('status@broadcast', {
    status: {
        text: "📢 Pengumuman penting untuk anggota grup kita!",
        color: '#FF1E3A5F', // Warna biru gelap
        font: 1,
        audience: 'group' // Otomatis menjadi Group Status!
    }
}, {
    statusJidList: participantJids
})
```

#### Contoh 4: Status dengan Gambar
```typescript
await sock.sendMessage('status@broadcast', {
    status: {
        image: { url: 'https://contoh.com/gambar.jpg' }, // Bisa juga buffer
        audience: 'all'
    }
})
```

---

## 3. proxyMetaAI

### Apa itu proxyMetaAI?
Fitur paling *gila* di fork ini. 😈

`sock.proxyMetaAI()` adalah sebuah fungsi bawaan yang menjadikan bot kamu sebagai **perantara (makelar/proxy)** antara pengguna di grup dengan **Meta AI resmi WhatsApp** — tanpa harus membayar API manapun!

### Cara Kerja Internal (Di Balik Layar)
```
Pengguna Grup  →  ".meta apa itu AI?"
       ↓
Baileys mengirim pesan ke nomor Meta AI resmi (DM diam-diam)
       ↓
Meta AI resmi membalas (streaming, kata per kata)
       ↓
Baileys menyadap setiap balasan dan LANGSUNG meng-edit pesan di grup
       ↓
Pengguna grup melihat efek "mengetik" layaknya Meta AI asli ✨
       ↓
Pesan final dibungkus dengan UI abu-abu khas Meta AI (fitur 'rich')
```

### Syarat Wajib
> [!IMPORTANT]
> Nomor WhatsApp yang terhubung ke Baileys instance ini **HARUS** sudah mendapatkan akses Meta AI secara resmi dari WhatsApp. Jika kamu membuka nomor bot di WA biasa dan bisa chat dengan Meta AI, maka fitur ini akan bekerja.

### Signature Fungsi

```typescript
sock.proxyMetaAI(
    query: string,        // Pertanyaan/teks yang ingin dikirim ke Meta AI
    targetJid?: string,   // JID tujuan pengiriman hasil (grup atau nomor pribadi)
    options?: {
        onProgress?: (text: string) => void,  // Callback setiap kali ada teks baru
        timeoutMs?: number                     // Batas waktu tunggu (default: 30000ms)
    }
): Promise<string>        // Mengembalikan teks jawaban akhir
```

### Contoh Kode Lengkap

#### Contoh 1: Implementasi Dasar (.meta command di Grup)
```typescript
sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0]
    if (!msg.message) return
    
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text || ""
    const sender = msg.key.remoteJid!

    if (text.toLowerCase().startsWith('.meta ')) {
        const query = text.slice(6) // Hapus ".meta " dari depan
        
        await sock.proxyMetaAI(query, sender)
        // Selesai! Semua sudah ditangani otomatis oleh Baileys.
    }
})
```

#### Contoh 2: Dengan Callback Progres & Timeout Custom
```typescript
sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0]
    if (!msg.message) return
    
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text || ""
    const sender = msg.key.remoteJid!

    if (text.toLowerCase().startsWith('.meta ')) {
        const query = text.slice(6)
        
        const jawaban = await sock.proxyMetaAI(query, sender, {
            timeoutMs: 60000, // Tunggu maksimal 60 detik
            onProgress: (partialText) => {
                // Ini dipanggil setiap kali Meta AI mengirim update
                console.log('[Meta AI Sedang Mengetik]:', partialText.slice(0, 50) + '...')
            }
        })
        
        console.log('[Meta AI Selesai]:', jawaban)
        
        // Simpan ke database atau lakukan apapun dengan hasilnya
    }
})
```

#### Contoh 3: Tanpa Streaming (Hanya Ambil Teks)
```typescript
// Tidak perlu mengirim hasilnya ke grup, cukup ambil teksnya saja
const hasilAI = await sock.proxyMetaAI(
    "Apa ibu kota Indonesia?",
    undefined, // Tidak ada targetJid = tidak ada pesan yang dikirim ke grup
    { timeoutMs: 15000 }
)

console.log("Jawaban Meta AI:", hasilAI)
// Output: "Jawaban Meta AI: Ibu kota Indonesia adalah Jakarta..."
```

#### Contoh 4: Integrasi dengan Perintah Lain
```typescript
const COMMANDS = {
    '.meta': async (query: string, jid: string) => {
        return await sock.proxyMetaAI(query, jid)
    },
    '.ai': async (query: string, jid: string) => {
        return await sock.proxyMetaAI(query, jid)
    },
    '.tanya': async (query: string, jid: string) => {
        return await sock.proxyMetaAI(query, jid)
    }
}

sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0]
    if (!msg.message || msg.key.fromMe) return
    
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text || ""
    const jid = msg.key.remoteJid!
    
    for (const [prefix, handler] of Object.entries(COMMANDS)) {
        if (text.toLowerCase().startsWith(prefix + ' ')) {
            const query = text.slice(prefix.length + 1)
            await handler(query, jid)
            break
        }
    }
})
```

---

## 🛠️ Instalasi & Penggunaan Fork Ini

### Install via npm (from GitHub)
```bash
npm install github:paikynie/Baileys
```

### Atau gunakan bun
```bash
bun add github:paikynie/Baileys
```

### Import di kode kamu
```typescript
import makeWASocket, { DisconnectReason, useMultiFileAuthState } from 'Baileys'

const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')

const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true
})

sock.ev.on('creds.update', saveCreds)

// Sekarang kamu bisa gunakan:
// sock.sendMessage(jid, { rich: { ... } })
// sock.sendMessage('status@broadcast', { status: { ... } })
// sock.proxyMetaAI(query, jid)
```

---

## 📝 Changelog

| Versi | Tanggal | Perubahan |
|-------|---------|-----------|
| 1.0.0 | 2026-08-01 | Initial fork dari `WhiskeySockets/Baileys` |
| 1.1.0 | 2026-08-01 | Fix WhatsApp Username Bug (#2742) |
| 1.2.0 | 2026-08-01 | Tambah fitur AIRich (InteractiveMessage bypass) |
| 1.3.0 | 2026-08-01 | Tambah Advanced Status (Close Friends, Group Status, Custom BG) |
| 1.4.0 | 2026-08-01 | Tambah `proxyMetaAI` (Meta AI Proxy dengan streaming thinking) |

---

## ⚠️ Disclaimer

> Fork ini dibuat untuk **tujuan edukasi dan penelitian** tentang protokol WhatsApp Web API. Penggunaan fitur-fitur di atas adalah tanggung jawab pengguna masing-masing. Pastikan penggunaan kamu sesuai dengan [Ketentuan Layanan WhatsApp](https://www.whatsapp.com/legal/terms-of-service).

---

*Made with ❤️ by Tuan Pai & powered by Antigravity AI*
