<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f2027,50:203a43,100:2c5364&height=200&section=header&text=MEGATORU&fontSize=72&fontColor=ffffff&fontAlignY=38&desc=Modified%20Baileys%20Library&descAlignY=58&descColor=7ecbf7&animation=fadeIn" width="100%"/>

<br/>

[![Version](https://img.shields.io/badge/version-1.0.0-7ecbf7?style=for-the-badge&logo=semver&logoColor=white)](https://github.com/anse-ux/megatoru)
[![Base](https://img.shields.io/badge/base-Baileys%209.x-00b4d8?style=for-the-badge&logo=whatsapp&logoColor=white)](https://github.com/WhiskeySockets/Baileys)
[![Fork](https://img.shields.io/badge/fork-Ryzewa-0077b6?style=for-the-badge&logo=git&logoColor=white)](https://github.com/Davizuni17/Ryzewa)
[![License](https://img.shields.io/badge/license-MIT-90e0ef?style=for-the-badge)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D20.0.0-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org)

<br/>

> **Megatoru** es un fork modificado de [Ryzewa](https://github.com/Davizuni17/Ryzewa), basado en [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys) v9.  
> Optimizado para bots de WhatsApp con soporte nativo de botones interactivos, listas, álbumes y resolución de LID.

<br/>

[![Bot Demo](https://img.shields.io/badge/▶%20%20Prueba%20el%20Bot%20con%20base--toru-25D366?style=for-the-badge&logo=whatsapp&logoColor=white&labelColor=128C7E)](https://github.com/anse-ux/base-toru)

</div>

---


### 👋🏻 **Visita el repositorio de** `base-toru`, **todavía en condiciones beta y arreglo de errores.**
> Puedes dar una estrella para que pueda ir actualizando de a poco agregando más funciones.

------

### 🔑 **Puedes leer la guía de** `[![Guía](https://img.shields.io/badge/funciones-blue?style=for-the-badge)](guia.md)` **para que puedas saber un poco más.**

------

## 📍 ¿Qué es Megatoru?

**Megatoru** no es un bot. Es la **librería base** sobre la que se construye el bot.

Es un fork de [Ryzewa (Davizuni17)](https://github.com/Davizuni17/Ryzewa) — que a su vez es un fork de [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys) — con las siguientes diferencias clave:

| Característica | Baileys original | Megatoru |
|---|:---:|:---:|
| Botones interactivos | ❌ deprecados | ✅ funcionales |
| `sendList` nativo | ❌ | ✅ |
| `sendAlbum` | ❌ | ✅ |
| Resolución `@lid` | parcial | ✅ completa |
| Wrapper ESM/CJS dual | ❌ | ✅ |
| `makeInMemoryStore` | eliminado en v7 | ✅ restaurado |
| Envío a canales/newsletters | ❌ | ✅ |

---

## 📦 Instalación

### En tu bot (desde GitHub, sin publicar en npm)

```json
// package.json de tu bot
{
  "dependencies": {
    "@whiskeysockets/baileys": "github:anse-ux/megatoru"
  }
}
```

```bash
npm install
```

### Importación

```js
// ESM (recomendado para base-toru)
import {
  makeWASocket,
  useMultiFileAuthState,
  DisconnectReason,
  proto,
} from '@whiskeysockets/baileys'
```

```js
// CommonJS (compatible)
const {
  makeWASocket,
  useMultiFileAuthState,
} = require('@whiskeysockets/baileys')
```

---

## 🗂️ Estructura del paquete

```
megatoru/
├── lib/
│   ├── index.js          ← Entry point CommonJS (compilado)
│   ├── Socket/           ← Lógica de conexión y envío
│   ├── Utils/            ← Serialización, parsers, helpers
│   ├── Types/            ← Tipos TypeScript (.d.ts)
│   ├── Store/            ← makeInMemoryStore restaurado
│   ├── Defaults/         ← Configuraciones por defecto
│   ├── WABinary/         ← Protocolo binario de WhatsApp
│   └── WAProto/          ← Definiciones protobuf
├── WAProto/              ← Proto compilados
└── package.json
```

---

## 🔌 ¿Necesito un `base.js` o `simple.js`?

**Sí.** Megatoru es solo la capa de comunicación con WhatsApp.  
Para agregar métodos de alto nivel como `sendButton`, `sendAlbum`, `sendList`, `parseMention`, `downloadM`, etc., necesitas un archivo wrapper.

[**base-toru**](https://github.com/anse-ux/base-toru) incluye `lib/base.js` ya adaptado a megatoru.  
Si usas tu propio bot, crea un `simple.js` o `base.js` que importe de `@whiskeysockets/baileys` y extienda el socket con `Object.defineProperties`.

```js
// Ejemplo mínimo de base.js propio
import { default as _makeWASocket } from '@whiskeysockets/baileys'

export function makeWASocket(options) {
  const conn = _makeWASocket(options)
  // agrega tus métodos aquí
  return conn
}
```

---

## ✅ Compatibilidad

### Funciona con:

| Framework / Proyecto | Estado |
|---|:---:|
| [base-toru](https://github.com/anse-ux/base-toru) | ✅ Oficial |
| Proyectos ESM propios (Node ≥ 20) | ✅ |
| Proyectos CommonJS propios | ✅ |
| Proyectos con TypeScript | ✅ tipos incluidos |

### No compatible con:

| Proyecto | Razón |
|---|---|
| Bots Legacy (antes de multi-device) | WhatsApp ya no soporta el protocolo |
| `@adiwajshing/baileys` <5.x | API completamente diferente |
| Proyectos que dependen de `makeWALegacySocket` | Eliminado en Baileys v7+ |
| `whatsapp-web.js` (Selenium) | Librería distinta, no relacionada |

---

## ⚙️ Mecánicas importantes

### Conexión
Megatoru no cambia el flujo de conexión de Baileys. Se conecta igual:

```js
const { state, saveCreds } = await useMultiFileAuthState('./session')
const sock = makeWASocket({ auth: state, ... })
sock.ev.on('creds.update', saveCreds)
```

### Resolución de LID
WhatsApp introdujo identificadores `@lid` para usuarios en grupos.  
Megatoru resuelve automáticamente `@lid → @s.whatsapp.net` usando caché interno y `groupMetadata`.

```js
// En base.js de tu bot
const realJid = await jidLid.resolveLidToRealJid(groupId, conn)
```

### Botones interactivos
Los botones usan el protocolo `nativeFlowMessage` / `interactiveMessage`.  
Ver [guia.md](guia.md) para todos los ejemplos.

### Álbumes de media
Envío agrupado de imágenes y videos usando `albumMessage`:

```js
await sock.sendAlbum(jid, [
  { type: 'image', data: { url: 'https://...' }, caption: 'Foto 1' },
  { type: 'video', data: { url: 'https://...' }, caption: 'Video 1' },
])
```

---

## 📄 Dependencias principales

```json
{
  "libsignal":       "@meta.inc/libsignal",
  "pino":            "^9.x",
  "protobufjs":      "^7.x",
  "ws":              "^8.x",
  "fflate":          "^0.8.x",
  "lru-cache":       "^11.x",
  "axios":           "^1.x"
}
```

## 📜 Créditos

| Proyecto | Autor |
|---|---|
| [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys) | Rajeh & contributors |
| Megatoru | [anse-ux](https://github.com/anse-ux) |

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:2c5364,50:203a43,100:0f2027&height=120&section=footer" width="100%"/>

**Megatoru** — Modified Baileys · MIT License · [base-toru →](https://github.com/anse-ux/base-toru)

</div>
