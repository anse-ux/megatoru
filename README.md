<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:161b22,100:1f2937&height=200&section=header&text=megatoru&fontSize=72&fontColor=58a6ff&fontAlignY=38&desc=Modified%20Baileys%20Library&descAlignY=58&descColor=8b949e&animation=fadeIn" width="100%"/>

<br/>

[![Version](https://img.shields.io/badge/version-1.0.0-58a6ff?style=for-the-badge&logo=semver&logoColor=white)](https://github.com/anse-ux/megatoru)
[![Base](https://img.shields.io/badge/base-Baileys%209.x-1f6feb?style=for-the-badge&logo=whatsapp&logoColor=white)](https://github.com/WhiskeySockets/Baileys)
[![Fork of](https://img.shields.io/badge/fork_of-Ryzewa-30363d?style=for-the-badge&logo=git&logoColor=white)](https://github.com/Davizuni17/Ryzewa)
[![License](https://img.shields.io/badge/license-MIT-238636?style=for-the-badge)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D20.0.0-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![Module](https://img.shields.io/badge/module-CJS-f0db4f?style=for-the-badge&logo=javascript&logoColor=black)](https://nodejs.org)

<br/>

> Welcome to the `megatoru` repository.
> This is a modified fork of Baileys, tailored to provide a stable and customizable foundation for your WhatsApp bot implementations. You are free to adapt and configure it according to your requirements.

<br/>

[![View Guide](https://img.shields.io/badge/📖%20View%20Guide-161b22?style=for-the-badge&logoColor=white)](guia.md)
[![Try with base-toru](https://img.shields.io/badge/🤖%20Try%20with%20base--toru-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://github.com/anse-ux/base-toru)

</div>

---

## ⚡ What is megatoru?

**megatoru** is not a bot. It's the **base library** your bot is built on top of.

| Feature | Baileys original | megatoru |
|---|:---:|:---:|
| Interactive buttons | ❌ deprecated | ✅ working |
| `sendList` (single_select) | ❌ | ✅ |
| `sendAlbum` (grouped media) | ❌ | ✅ |
| `interactiveButtons` field | ❌ | ✅ |
| `@lid` resolution | partial | ✅ full |
| `makeInMemoryStore` | removed in v7 | ✅ restored |
| Newsletter / channel support | ❌ | ✅ |
| CJS + ESM compatible | CJS only | ✅ both |

---

## 📦 Installation

### As a dependency (recommended)

```json
// your bot's package.json
{
  "dependencies": {
    "@whiskeysockets/baileys": "github:anse-ux/megatoru"
  }
}
```

```bash
npm install
```

### Importing

```js
// ESM (recommended — used by base-toru)
import {
  makeWASocket,
  useMultiFileAuthState,
  DisconnectReason,
  proto,
} from '@whiskeysockets/baileys'

// CommonJS
const {
  makeWASocket,
  useMultiFileAuthState,
} = require('@whiskeysockets/baileys')
```

---

## ⚠️ Critical: How to import `makeWASocket` correctly

This is the most common mistake when wrapping megatoru in a `base.js` or `simple.js`.

megatoru compiles to **CommonJS** (`lib/index.js`). When Node.js loads a CJS package from ESM, the entire `module.exports` object is available — but **`.default` is NOT a callable function**.

### ❌ Wrong — will throw `TypeError: baileys.default is not a function`

```js
import * as baileys from '@whiskeysockets/baileys'

// This fails — baileys.default is the module.exports object, not a function
const conn = baileys.default(connectionOptions)
```

```js
// Also wrong
const { default: _makeWASocket } = (await import('@whiskeysockets/baileys'))
// Calling _makeWASocket() will fail for the same reason
```

```js
// Also wrong — old pattern from Baileys v5/v6
const { default: _makeWASocket } = (await import('@whiskeysockets/baileys')).default
```

### ✅ Correct — use named exports directly

megatoru exports `makeWASocket` as a **named export** in `lib/index.js`:

```js
exports.makeWASocket = Socket_1.default  // named export ✅
exports.default = Socket_1.default       // also available, but NOT as a callable default
```

So the correct import is:

```js
// ESM — named import
import {
  makeWASocket as _makeWASocket,
  proto,
  useMultiFileAuthState,
  DisconnectReason,
  generateWAMessageFromContent,
  downloadContentFromMessage,
  jidDecode,
  areJidsSameUser,
  WAMessageStubType,
  extractMessageContent,
  prepareWAMessageMedia,
  delay,
} from '@whiskeysockets/baileys'

// Then use it:
const conn = _makeWASocket(connectionOptions)  // ✅ works
```

```js
// CommonJS — also fine
const { makeWASocket } = require('@whiskeysockets/baileys')
const conn = makeWASocket(connectionOptions)  // ✅ works
```

### Writing your own `base.js` / `simple.js`

```js
// lib/base.js — minimal correct wrapper
import {
  makeWASocket as _makeWASocket,   // ✅ named import of the default export
  proto,
  generateWAMessageFromContent,
  // ...other named exports
} from '@whiskeysockets/baileys'

export function makeWASocket(connectionOptions, options = {}) {
  const conn = _makeWASocket(connectionOptions)  // ✅ call the named export

  // extend the socket with your own methods
  Object.defineProperties(conn, {
    Reply: {
      value(jid, text, quoted) {
        return conn.sendMessage(jid, { text }, { quoted })
      },
      enumerable: true,
    },
    // ... more methods
  })

  return conn
}
```

> **Note:** `base-toru` ships a full `lib/base.js` already adapted to megatoru.  
> If you're building your own bot, use it as a reference.

---

## 🗂️ Package structure

```
megatoru/
├── lib/
│   ├── index.js          ← CJS entry point (compiled)
│   ├── Socket/           ← Connection logic, message sending
│   ├── Utils/            ← Serialization, parsers, helpers
│   ├── Types/            ← TypeScript definitions (.d.ts)
│   ├── Store/            ← makeInMemoryStore (restored)
│   ├── Defaults/         ← Default configurations
│   ├── WABinary/         ← WhatsApp binary protocol
│   └── WAProto/          ← Compiled protobuf definitions
├── WAProto/              ← Proto files
└── package.json
```

---

## ✅ Compatibility

### Works with

| Project / Setup | Status |
|---|:---:|
| [base-toru](https://github.com/anse-ux/base-toru) | ✅ Official |
| Custom ESM bots (Node ≥ 20) | ✅ |
| Custom CJS bots | ✅ |
| TypeScript projects | ✅ types included |

### Does NOT work with

| Project | Reason |
|---|---|
| Legacy Baileys (<5.x) | Completely different API |
| `makeWALegacySocket` dependents | Removed in Baileys v7+ |
| `whatsapp-web.js` | Different library, not related |
| Bots calling `baileys.default()` | See import guide above |

---

## 🔑 Key exports

```js
import {
  makeWASocket,               // Main socket factory
  useMultiFileAuthState,      // Session manager (credentials)
  makeCacheableSignalKeyStore,// Key store with cache
  fetchLatestBaileysVersion,  // Get latest WA version
  DisconnectReason,           // Disconnect reason codes
  proto,                      // WhatsApp protobuf definitions
  generateWAMessage,          // Generate WA message object
  generateWAMessageFromContent,
  generateForwardMessageContent,
  downloadContentFromMessage, // Download media from message
  jidDecode,                  // Decode multi-device JID
  jidNormalizedUser,          // Normalize JID to standard form
  areJidsSameUser,            // Compare two JIDs
  WAMessageStubType,          // Stub type constants
  extractMessageContent,      // Extract message content
  prepareWAMessageMedia,      // Prepare media for upload
  getContentType,             // Get message content type
  delay,                      // Promise-based delay
} from '@whiskeysockets/baileys'
```

---

## 📜 Credits

| Project | Author |
|---|---|
| [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys) | Rajeh & contributors |
| [Ryzewa](https://github.com/Davizuni17/Ryzewa) | Davizuni17 |
| megatoru | [anse-ux](https://github.com/anse-ux) |

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:1f2937,50:161b22,100:0d1117&height=120&section=footer" width="100%"/>

**megatoru** — Modified Baileys · MIT License

[![anse-ux](https://img.shields.io/badge/developer-anse--ux-58a6ff?style=flat-square&logo=github&logoColor=white)](https://github.com/anse-ux)
[![base-toru](https://img.shields.io/badge/bot_base-base--toru-25D366?style=flat-square&logo=whatsapp&logoColor=white)](https://github.com/anse-ux/base-toru)
[![Guide](https://img.shields.io/badge/guide-guia.md-1f6feb?style=flat-square)](guia.md)

</div>
