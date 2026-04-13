<div align="center">

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0f2027,100:2c5364&height=80&text=MEGATORU%20·%20Guía%20de%20uso&fontSize=28&fontColor=7ecbf7&fontAlignY=55" width="100%"/>

</div>

# 📖 Guía completa de Megatoru

> Referencia de todos los métodos disponibles en Megatoru a través de `lib/base.js` (base-toru) o tu propio wrapper.  
> Todos los ejemplos asumen que tienes `sock` como el socket extendido.

---

## Índice

- [Mensajes de texto](#-mensajes-de-texto)
- [Archivos y media](#-archivos-y-media)
- [Botones interactivos](#-botones-interactivos)
- [Listas interactivas](#-listas-interactivas)
- [Álbumes de media](#-álbumes-de-media)
- [Encuestas](#-encuestas)
- [Contactos](#-contactos)
- [AdReply (preview externo)](#-adreply-preview-externo)
- [Mensajes de evento](#-mensajes-de-evento)
- [Grupos — gestión de usuarios](#-grupos--gestión-de-usuarios)
- [Anti-spam y cooldown](#-anti-spam-y-cooldown)
- [Menciones y LID](#-menciones-y-lid)
- [Descargar media](#-descargar-media)
- [Reacciones](#-reacciones)
- [Canales / Newsletters](#-canales--newsletters)

---

## 💬 Mensajes de texto

```js
// Texto simple
await sock.sendMessage(jid, { text: 'Hola mundo' })

// Texto con mención
await sock.sendMessage(jid, {
  text: '@521234567890 hola!',
  mentions: ['521234567890@s.whatsapp.net'],
})

// Respuesta rápida (alias sock.Reply)
await sock.Reply(jid, 'Texto de respuesta', m)

// Editar un mensaje enviado
const sent = await sock.Reply(jid, 'Texto original', m)
await sock.sendMessage(jid, { text: 'Texto editado', edit: sent.key })

// Eliminar un mensaje
await sock.sendMessage(jid, { delete: m.key })
```

---

## 📁 Archivos y media

```js
// Imagen desde URL
await sock.sendMessage(jid, {
  image: { url: 'https://ejemplo.com/foto.jpg' },
  caption: 'Pie de foto',
}, { quoted: m })

// Video
await sock.sendMessage(jid, {
  video: { url: 'https://ejemplo.com/video.mp4' },
  caption: 'Mi video',
  mimetype: 'video/mp4',
})

// Audio (PTT = nota de voz)
await sock.sendMessage(jid, {
  audio: { url: 'https://ejemplo.com/audio.mp3' },
  mimetype: 'audio/mpeg',
  ptt: true, // nota de voz
})

// Documento / archivo
await sock.sendMessage(jid, {
  document: { url: 'https://ejemplo.com/archivo.pdf' },
  mimetype: 'application/pdf',
  fileName: 'documento.pdf',
})

// Sticker desde buffer
await sock.sendMessage(jid, {
  sticker: fs.readFileSync('./sticker.webp'),
})

// Envío automático con detección de tipo (alias sock.Files)
await sock.Files(jid, 'https://ejemplo.com/media.mp4', 'video.mp4', 'Caption', m)
await sock.Files(jid, buffer, 'audio.ogg', '', m, true) // ptt=true
```

---

## 🔘 Botones interactivos

Los botones usan `nativeFlowMessage` con `viewOnceMessage`.  
Tipos disponibles: `quick_reply`, `cta_url`, `cta_copy`.

```js
// Botones simples (quick_reply)
await sock.sendButton(
  jid,
  'Elige una opción',       // texto del body
  'base-toru',              // footer
  'https://i.imgur.com/x.jpg', // imagen (URL o Buffer, opcional)
  [
    ['Opción 1', 'id_opcion1'],
    ['Opción 2', 'id_opcion2'],
    ['Opción 3', 'id_opcion3'],
  ],
  null,                     // copy (texto a copiar, opcional)
  null,                     // urls (botones de URL, opcional)
  m                         // quoted
)

// Botón de copiar texto
await sock.sendButton(
  jid,
  'Copia este código',
  '',
  null,
  [],
  'CODIGO-ABC123',          // se copia al presionar
  null,
  m
)

// Botones + URL
await sock.sendButton(
  jid,
  'Visita nuestra web',
  '',
  null,
  [['Ir al menú', 'cmd_menu']],
  null,
  [
    ['🌐 Sitio web', 'https://github.com/anse-ux/base-toru'],
    ['💬 Grupo',     'https://chat.whatsapp.com/xxx'],
  ],
  m
)
```

### Recibir respuesta de botón

```js
// En tu handler/plugin
if (m.mtype === 'interactiveResponseMessage') {
  const response = JSON.parse(m.msg?.nativeFlowResponseMessage?.paramsJson || '{}')
  const id = response.id // el id del botón presionado
  console.log('Botón presionado:', id)
}
```

---

## 📋 Listas interactivas

```js
await sock.sendList(
  jid,
  'Título del menú',        // header
  'Selecciona una opción',  // body
  '📋 Ver opciones',        // texto del botón
  [
    {
      title: '🎵 Música',
      rows: [
        { title: '!spotify',   description: 'Buscar en Spotify', id: '!spotify' },
        { title: '!play',      description: 'Reproducir canción', id: '!play' },
      ],
    },
    {
      title: '📥 Descargas',
      rows: [
        { title: '!tiktok',    description: 'Descargar TikTok', id: '!tiktok' },
        { title: '!yt',        description: 'Descargar YouTube', id: '!yt' },
      ],
    },
  ],
  m                         // quoted
)
```

### Recibir selección de lista

```js
if (m.mtype === 'interactiveResponseMessage') {
  const data = JSON.parse(m.msg?.nativeFlowResponseMessage?.paramsJson || '{}')
  const selected = data.id // el id de la fila seleccionada
}
```

---

## 🖼️ Álbumes de media

Envía múltiples imágenes y/o videos agrupados en un álbum.  
Mínimo 2 medias. Máximo recomendado: 10.

```js
await sock.sendAlbum(
  jid,
  [
    { type: 'image', data: { url: 'https://i.imgur.com/a.jpg' }, caption: 'Foto 1' },
    { type: 'image', data: { url: 'https://i.imgur.com/b.jpg' }, caption: 'Foto 2' },
    { type: 'video', data: { url: 'https://ejemplo.com/c.mp4' }, caption: 'Video' },
  ],
  {
    quoted: m,      // mensaje citado (opcional)
    delay: 500,     // ms entre cada media (default: 500)
  }
)

// Con buffer local
await sock.sendAlbum(jid, [
  { type: 'image', data: fs.readFileSync('./foto1.jpg') },
  { type: 'image', data: fs.readFileSync('./foto2.jpg') },
])
```

---

## 📊 Encuestas

```js
await sock.sendPoll(
  jid,
  '¿Cuál es tu lenguaje favorito?',
  [
    ['JavaScript'],
    ['Python'],
    ['TypeScript'],
    ['Rust'],
  ]
)
```

### Leer resultado de encuesta

```js
import { getAggregateVotesInPollMessage } from '@whiskeysockets/baileys'

sock.ev.on('messages.update', async updates => {
  for (const { key, update } of updates) {
    if (update.pollUpdates) {
      const pollMsg   = await sock.loadMessage(key.remoteJid, key.id)
      const votes     = getAggregateVotesInPollMessage({ message: pollMsg, pollUpdates: update.pollUpdates })
      console.log('Votos:', votes)
    }
  }
})
```

---

## 📇 Contactos

```js
// Contacto simple
await sock.sendContact(
  jid,
  [['521234567890', 'Juan Pérez']],
  m
)

// Múltiples contactos
await sock.sendContact(
  jid,
  [
    ['521234567890', 'Juan'],
    ['549123456789', 'Ana'],
  ],
  m
)
```

---

## 🔗 AdReply (preview externo)

Mensaje de texto con tarjeta de preview personalizada (título, imagen, URL).

```js
await sock.sendToruWa(
  jid,
  'Texto del mensaje',
  bufferImagen,           // Buffer de la imagen thumbnail
  'Título de la tarjeta',
  'Subtítulo / body',
  'https://enlace.com',  // URL de la tarjeta
  m                      // quoted
)

// O directo con generateWAMessageFromContent
import { generateWAMessageFromContent } from '@whiskeysockets/baileys'

const msg = generateWAMessageFromContent(jid, {
  extendedTextMessage: {
    text: 'Mi texto',
    contextInfo: {
      externalAdReply: {
        title:                  'Mi título',
        body:                   'Descripción',
        thumbnail:              buffer,
        sourceUrl:              'https://ejemplo.com',
        mediaType:              1,
        renderLargerThumbnail:  false,
      },
    },
  },
}, { quoted: m })

await sock.relayMessage(jid, msg.message, { messageId: msg.key.id })
```

---

## 📅 Mensajes de evento

```js
await sock.sendEvent(
  jid,
  'Nombre del evento',
  'Descripción del evento',
  'Ciudad, País',           // nombre de ubicación
  'https://link-reunion.com'
)
```

---

## 👥 Grupos — gestión de usuarios

```js
// Agregar participantes
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'add')

// Eliminar participantes
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'remove')

// Promover a admin
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'promote')

// Quitar admin
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'demote')

// Obtener metadata del grupo
const meta = await sock.groupMetadata(jid)
console.log(meta.subject)       // nombre del grupo
console.log(meta.participants)  // lista de participantes
console.log(meta.desc)          // descripción

// Cambiar nombre del grupo
await sock.groupUpdateSubject(jid, 'Nuevo nombre')

// Cambiar descripción
await sock.groupUpdateDescription(jid, 'Nueva descripción')

// Obtener link de invitación
const link = await sock.groupInviteCode(jid)
console.log(`https://chat.whatsapp.com/${link}`)

// Revocar link de invitación
await sock.groupRevokeInvite(jid)

// Solo admins pueden enviar mensajes
await sock.groupSettingUpdate(jid, 'announcement')

// Todos pueden enviar mensajes
await sock.groupSettingUpdate(jid, 'not_announcement')

// Bloquear edición de info del grupo a no-admins
await sock.groupSettingUpdate(jid, 'locked')
await sock.groupSettingUpdate(jid, 'unlocked')
```

---

## 🛡️ Anti-spam y cooldown

Patrón recomendado en tus plugins:

```js
// En tu plugin
handler.before = async (m, { conn }) => {
  const user = global.db.data.users[m.sender]

  // Cooldown de 5 segundos por usuario
  const lastCmd = user.lastCmd || 0
  if (Date.now() - lastCmd < 5000) {
    await conn.Reply(m.chat, '⏳ Espera un momento antes de usar otro comando.', m)
    return true // detiene la ejecución del plugin
  }

  user.lastCmd = Date.now()
  return false
}

// O usando el sistema del handler (spam integrado en handler.js)
// El handler.js ya aplica un cooldown de 3s automático a todos los usuarios.
// Para comandos específicos con cooldown mayor:
handler.exp = 0 // no dar XP en este comando
```

---

## 🏷️ Menciones y LID

```js
// parseMention — convierte @número en JID
const mentions = sock.parseMention('@521234567890 hola')
// → ['521234567890@s.whatsapp.net']

// Resolver LID a JID real (en grupos)
const realJid = await '2064123456@lid'.resolveLidToRealJid(groupJid, sock)
// → '521234567890@s.whatsapp.net'

// Enviar mención
await sock.sendMessage(jid, {
  text: 'Hola @521234567890',
  mentions: ['521234567890@s.whatsapp.net'],
})
```

---

## 📥 Descargar media

```js
// Desde un mensaje recibido
const buffer = await m.download()

// Especificando tipo
const buffer = await sock.downloadM(m.msg, m.mediaType.replace('Message', ''))

// Guardando a archivo
const filePath = await sock.downloadM(m.msg, 'image', true)
console.log(filePath) // → './tmp/1234567890.jpg'
```

---

## 👍 Reacciones

```js
await sock.sendMessage(jid, {
  react: {
    text: '🔥',    // emoji de reacción
    key:  m.key,   // key del mensaje al que reaccionar
  },
})

// Quitar reacción
await sock.sendMessage(jid, {
  react: { text: '', key: m.key }
})
```

---

## 📡 Canales / Newsletters

```js
// Seguir un canal
await sock.newsletterFollow('120363424098891946@newsletter')

// Dejar de seguir
await sock.newsletterUnfollow('120363424098891946@newsletter')

// Enviar mensaje a un canal (requiere ser admin del canal)
await sock.sendMessage('120363424098891946@newsletter', {
  text: 'Actualización del canal',
})

// Obtener metadata del canal
const meta = await sock.newsletterMetadata('120363424098891946@newsletter')
console.log(meta.name)
console.log(meta.subscribers)
```

---

<div align="center">

¿Falta algún método? Abre un [issue](https://github.com/anse-ux/megatoru/issues) o contribuye con un PR.

**[← README](README.md)** · **[base-toru →](https://github.com/anse-ux/base-toru)**

</div>
