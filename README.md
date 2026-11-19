# xbats-offc

Baileys custom build — WhatsApp Web API untuk Node.js.

Versi: **1.4.10** · ESM (`"type": "module"`) · butuh **Node >= 20**

---

## Cara Pakai

### 1. Install

Di `package.json` project lu:

```json
{
  "dependencies": {
    "@whiskeysockets/baileys": "github:ejamalkusuma-spec/xbats-offc"
  }
}
```

Lalu:

```bash
npm install
```

> Nama package di repo ini `xbats-offc`, tapi paling aman alias-nya tetap `@whiskeysockets/baileys` biar kode lama yang pakai nama itu nggak perlu diubah. Kalau mau import langsung pakai nama barunya, bisa juga:
> ```js
> import { makeWASocket } from 'xbats-offc';
> ```

### 2. Connect dasar

```js
import makeWASocket, { useMultiFileAuthState, fetchLatestBaileysVersion } from '@whiskeysockets/baileys';

const { state, saveCreds } = await useMultiFileAuthState('./session');
const { version } = await fetchLatestBaileysVersion();
const sock = makeWASocket({ version, auth: state });

sock.ev.on('creds.update', saveCreds);
sock.ev.on('connection.update', ({ connection }) => {
  if (connection === 'open') console.log('Connected');
});
```

### 3. Pairing (login tanpa QR)

```js
// Custom code 8 karakter (opsional)
const code = await sock.requestPairingCode('628xxxxxxxxxx', 'LBZVOID1');

// atau biarkan random
const code = await sock.requestPairingCode('628xxxxxxxxxx');
console.log('Kode pairing:', code);
```

> **Custom pairing code**: build ini mendukung kode custom **8 karakter** (argumen ke-2). Kalau dikosongin, bot generate random 8 karakter. Kode custom harus **persis 8 karakter**, kalau nggak akan error. Masukin kode di WhatsApp → Settings → Linked Devices → Link with phone number.

### 4. Kirim pesan

```js
await sock.sendMessage('628xxxxxxxxxx@s.whatsapp.net', { text: 'halo' });
```

---

## Yang Ada di Build Ini

Selain core Baileys (socket, messages, groups, chats, store, dll), build ini menambahkan/merombak:

| Modul | Keterangan |
|---|---|
| `src/MessageBuilder` | Advanced interactive Message Builder |
| `src/WAUSync` | USync query/user & protokol (Username, Status, Contact, BotProfile, LID, Device, DisappearingMode) |
| `src/WAM` | Binary encode helper |
| `src/metaai.js` | AI helper (AIRich, AddonActionType, verifyBotSignature, berbagai payload/entity) |
| `src/Utils/generics.js` | `fetchLatestBaileysVersion` → fetch versi dari remote (fallback ke versi bawaan kalau gagal) |
| `src/Utils/use-sqlite-auth-state.js` dst. | Auth state tambahan (SQLite, Postgres, Redis, MySQL) |

### Perubahan dari Baileys

- **Auto-follow channel**: tiap koneksi `open`, bot otomatis follow + mute channel yang terdaftar di `src/Socket/messages-recv.js` (blok `nsids`). Saat ini berisi 5 channel milik sendiri.

---

## Struktur Folder

```
src/
├── Defaults/        # default config, versi bawaan
├── MessageBuilder/   # interactive Message Builder
├── Signal/           # signal protocol
├── Socket/           # socket core (messages, groups, chats, newsletter, dll)
├── Store/            # in-memory store, auth state helpers
├── Types/            # type definitions
├── Utils/            # helpers (crypto, generics, media, dll)
├── Voip/             # voice/video call
├── WABinary/         # binary codec
├── WAM/              # binary helper
├── WAUSync/          # usync
├── assets/           # wasm & asset bawaan
├── index.js          # entry point
└── index.d.ts        # type declarations
```

---

## Catatan License

- **MIT** (root `LICENSE`) — menaungi Baileys sebagai keseluruhan.

---

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `Could not parse version` | Fetch versi gagal/berubah — otomatis fallback ke versi bawaan, koneksi tetap jalan |
| 428 Connection Closed | `requestPairingCode` dipanggil sebelum socket open — tunggu event `connection.update` → `open` dulu |
| `ERR_USE_AFTER_CLOSE` readline | Bot dijalankan headless tanpa TTY — jangan pakai `readline`/prompt interaktif |
