<div align="center">

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0d1117,100:161b22&height=80&text=megatoru%20·%20Guide&fontSize=28&fontColor=58a6ff&fontAlignY=55" width="100%"/>

</div>

# 📖 megatoru — Complete Guide

> Reference for all message types and button structures supported by megatoru.  

---

## Table of Contents

- [Text messages](#-text-messages)
- [Media files](#-media-files)
- [Simple buttons](#-simple-buttons-type-1)
- [Flow buttons (mixed)](#-flow-buttons-mixed)
- [Interactive buttons — all types](#-interactive-buttons--all-types)
- [Interactive with image/video header](#-interactive-with-imagevideo-header)
- [List messages](#-list-messages)
- [Albums](#-albums)
- [Polls](#-polls)
- [AdReply (external preview)](#-adreply-external-preview)
- [Contacts](#-contacts)
- [Events](#-events)
- [Reactions](#-reactions)
- [Newsletters / Channels](#-newsletters--channels)
- [Group management](#-group-management)
- [Compatibility notes](#-compatibility-notes)

---

## 💬 Text messages

```js
// Plain text
await sock.sendMessage(jid, { text: 'Hello!' })

// With mention
await sock.sendMessage(jid, {
  text: '@521234567890 hello!',
  mentions: ['521234567890@s.whatsapp.net'],
})

// Edit a sent message
const sent = await sock.sendMessage(jid, { text: 'Original' })
await sock.sendMessage(jid, { text: 'Edited', edit: sent.key })

// Delete a message
await sock.sendMessage(jid, { delete: m.key })
```

---

## 📁 Media files

```js
// Image from URL
await sock.sendMessage(jid, {
  image: { url: 'https://example.com/photo.jpg' },
  caption: 'Caption text',
}, { quoted: m })

// Image from buffer
await sock.sendMessage(jid, {
  image: fs.readFileSync('./photo.jpg'),
  caption: 'Caption',
})

// Video
await sock.sendMessage(jid, {
  video: { url: 'https://example.com/video.mp4' },
  caption: 'Video caption',
  mimetype: 'video/mp4',
})

// Audio (voice note)
await sock.sendMessage(jid, {
  audio: { url: 'https://example.com/audio.mp3' },
  mimetype: 'audio/mpeg',
  ptt: true,
})

// Document
await sock.sendMessage(jid, {
  document: { url: 'https://example.com/file.pdf' },
  mimetype: 'application/pdf',
  fileName: 'document.pdf',
})

// Sticker
await sock.sendMessage(jid, {
  sticker: fs.readFileSync('./sticker.webp'),
})
```

---

## 🔘 Simple buttons (type 1)

The most basic button type. Uses `buttons` field with `buttonId` and `buttonText`.

```js
await sock.sendMessage(jid, {
  text: 'Choose an option:',
  footer: 'megatoru Bot',
  buttons: [
    {
      buttonId: 'btn_menu',
      buttonText: { displayText: '📋 Menu' },
      type: 1,
    },
    {
      buttonId: 'btn_ping',
      buttonText: { displayText: '⚡ Ping' },
      type: 1,
    },
    {
      buttonId: 'btn_info',
      buttonText: { displayText: '👤 Info' },
      type: 1,
    },
  ],
  headerType: 1,
  viewOnce:   true,
}, { quoted: m })
```

### With image header

```js
await sock.sendMessage(jid, {
  image: { url: 'https://example.com/banner.jpg' },
  caption: 'Choose an option:',
  footer: 'megatoru Bot',
  buttons: [
    { buttonId: 'btn_1', buttonText: { displayText: '✅ Accept' }, type: 1 },
    { buttonId: 'btn_2', buttonText: { displayText: '❌ Reject' }, type: 1 },
  ],
  headerType: 4,   // 4 = image header
  viewOnce:   true,
}, { quoted: m })
```

### Receiving button responses

```js
// In your handler/plugin
if (m.mtype === 'buttonsResponseMessage') {
  const id = m.msg?.selectedButtonId
  console.log('Button pressed:', id)
}
```

---

## 🔀 Flow buttons (mixed)

Combines regular `quick_reply` buttons with a native flow list (`single_select`) using `interactiveButtons`.

```js
await sock.sendMessage(jid, {
  text: '¡Control Panel!',
  title: '🤖 Main Panel',
  subtitle: 'Choose a category',
  footer: '© megatoru',
  interactiveButtons: [
    // Quick reply buttons
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({ display_text: '📋 Menu', id: '.menu' }),
    },
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({ display_text: '⚡ Ping', id: '.ping' }),
    },
    // Dropdown list
    {
      name: 'single_select',
      buttonParamsJson: JSON.stringify({
        title: '🔽 View Categories',
        sections: [
          {
            title: 'Tools',
            highlight_label: '🛠️',
            rows: [
              {
                header: '🔍',
                title: 'Google Search',
                description: 'Search the web',
                id: '.google',
              },
              {
                header: '🖼️',
                title: 'Create Sticker',
                description: 'Image to sticker',
                id: '.sticker',
              },
            ],
          },
          {
            title: 'Entertainment',
            highlight_label: '🎮',
            rows: [
              {
                header: '🎵',
                title: 'Download Music',
                description: 'From YouTube',
                id: '.play',
              },
            ],
          },
        ],
      }),
    },
  ],
  headerType: 1,
  viewOnce: true,
}, { quoted: m })
```

---

## 🎛️ Interactive buttons — all types

megatoru supports all `interactiveButtons` types via `name` + `buttonParamsJson`:

```js
await sock.sendMessage(jid, {
  text: 'Interactive message body',
  title: '🤖 Interactive Panel',
  subtitle: 'Choose an action',
  footer: 'megatoru © 2025',
  interactiveButtons: [

    // ── quick_reply: sends a text response ──────────────────
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({
        display_text: '✅ Accept',
        id: 'accept',
      }),
    },

    // ── cta_url: opens a URL ────────────────────────────────
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: '🌐 Visit Website',
        url: 'https://github.com/anse-ux/base-toru',
      }),
    },

    // ── cta_copy: copies text to clipboard ──────────────────
    {
      name: 'cta_copy',
      buttonParamsJson: JSON.stringify({
        display_text: '📋 Copy Code',
        id: 'copy_code',
        copy_code: 'MEGATORU2026',
      }),
    },

    // ── cta_call: initiates a phone call ────────────────────
    {
      name: 'cta_call',
      buttonParamsJson: JSON.stringify({
        display_text: '📞 Call',
        id: '+521234567890',
      }),
    },

    // ── single_select: dropdown list ────────────────────────
    {
      name: 'single_select',
      buttonParamsJson: JSON.stringify({
        title: '📜 View Options',
        sections: [
          {
            title: 'Settings',
            highlight_label: '⚙️',
            rows: [
              {
                header: 'Language',
                title: 'Change Language',
                description: 'Spanish / English',
                id: 'change_lang',
              },
              {
                header: 'Theme',
                title: 'Change Theme',
                description: 'Light / Dark',
                id: 'change_theme',
              },
            ],
          },
        ],
      }),
    },

    // ── send_location: request user location ────────────────
    {
      name: 'send_location',
      buttonParamsJson: '',
    },

  ],
  headerType: 1,
  viewOnce:   true,
}, { quoted: m })
```

### Receiving interactive responses

```js
if (m.mtype === 'interactiveResponseMessage') {
  const data = JSON.parse(
    m.msg?.nativeFlowResponseMessage?.paramsJson || '{}'
  )
  const id = data.id  // the id of the selected button/row
  console.log('Selected:', id)
}
```

---

## 🖼️ Interactive with image/video header

```js
// Image header
await sock.sendMessage(jid, {
  image: { url: 'https://example.com/banner.jpg' },
  caption: 'Body description',
  title: 'Message Title',
  subtitle: 'Subtitle',
  footer: 'megatoru',
  media: true,               // ← required for media header
  interactiveButtons: [
    {
      name: 'quick_reply',
      buttonParamsJson: JSON.stringify({ display_text: '👍 Like', id: 'like' }),
    },
    {
      name: 'cta_url',
      buttonParamsJson: JSON.stringify({
        display_text: '🔗 Learn more',
        url: 'https://github.com/anse-ux/base-toru',
      }),
    },
  ],
}, { quoted: m })

// Video header — same structure, replace image with video
await sock.sendMessage(jid, {
  video: { url: 'https://example.com/clip.mp4' },
  caption: 'Video description',
  media: true,
  interactiveButtons: [ /* ... */ ],
}, { quoted: m })
```

---

## 📋 List messages

Pure dropdown list using `single_select`. No other buttons.

```js
await sock.sendMessage(jid, {
  text: 'Select a category from the button below.',
  title: '📋 Options Menu',
  footer: 'megatoru',
  interactiveButtons: [
    {
      name: 'single_select',
      buttonParamsJson: JSON.stringify({
        title: '📋 View categories',
        sections: [
          {
            title: '🧩 Plugins',
            rows: [
              { header: '', title: '.menu', description: 'Main menu', id: '.menu' },
              { header: '', title: '.ping', description: 'Check latency', id: '.ping' },
              { header: '', title: '.infobot', description: 'Bot info', id: '.infobot' },
            ],
          },
          {
            title: '⚙️ System',
            rows: [
              { header: '', title: '.run', description: 'Bot uptime', id: '.run' },
              { header: '', title: '.bots', description: 'Active bots', id: '.bots' },
            ],
          },
        ],
      }),
    },
  ],
  headerType: 1,
  viewOnce:   true,
}, { quoted: m })
```

---

## 🖼️ Albums

Send grouped images and/or videos. Minimum 2 items.

```js
// URLs
await sock.sendAlbum(jid, [
  { type: 'image', data: { url: 'https://example.com/a.jpg' }, caption: 'Photo 1' },
  { type: 'image', data: { url: 'https://example.com/b.jpg' }, caption: 'Photo 2' },
  { type: 'video', data: { url: 'https://example.com/c.mp4' }, caption: 'Video'   },
], { quoted: m, delay: 500 })

// Buffers
await sock.sendAlbum(jid, [
  { type: 'image', data: fs.readFileSync('./photo1.jpg') },
  { type: 'image', data: fs.readFileSync('./photo2.jpg') },
])
```

---

## 📊 Polls

```js
await sock.sendPoll(jid, 'What is your favorite language?', [
  ['JavaScript'],
  ['Python'],
  ['TypeScript'],
  ['Rust'],
])
```

---

## 🔗 AdReply (external preview)

Message with a custom link preview card.

```js
import { generateWAMessageFromContent } from '@whiskeysockets/baileys'

const prep = generateWAMessageFromContent(jid, {
  extendedTextMessage: {
    text: 'Message text',
    contextInfo: {
      externalAdReply: {
        title: 'Card Title',
        body: 'Card description',
        thumbnail: imageBuffer,
        sourceUrl: 'https://example.com',
        mediaType: 1,
        renderLargerThumbnail: false,
      },
    },
  },
}, { quoted: m })

await sock.relayMessage(jid, prep.message, { messageId: prep.key.id })
```

---

## 📇 Contacts

```js
// Single contact
await sock.sendMessage(jid, {
  contacts: {
    displayName: 'John Doe',
    contacts: [{
      displayName: 'John Doe',
      vcard: `BEGIN:VCARD\nVERSION:3.0\nFN:John Doe\nTEL;waid=521234567890:+521234567890\nEND:VCARD`,
    }],
  },
}, { quoted: m })
```

---

## 📅 Events

```js
import { generateWAMessageFromContent } from '@whiskeysockets/baileys'

const msg = generateWAMessageFromContent(jid, {
  messageContextInfo: {},
  eventMessage: {
    isCanceled: false,
    name: 'Event Name',
    description: 'Event description',
    location: { degreesLatitude: 0, degreesLongitude: 0, name: 'City, Country' },
    joinLink: 'https://meet.example.com',
    startTime: String(Math.floor(Date.now() / 1000)),
  },
}, {})

await sock.relayMessage(jid, msg.message, { messageId: msg.key.id })
```

---

## 👍 Reactions

```js
// Add reaction
await sock.sendMessage(jid, {
  react: { text: '🔥', key: m.key },
})

// Remove reaction
await sock.sendMessage(jid, {
  react: { text: '', key: m.key },
})
```

---

## 📡 Newsletters / Channels

```js
// Follow a channel
await sock.newsletterFollow('120363424098891946@newsletter')

// Unfollow
await sock.newsletterUnfollow('120363424098891946@newsletter')

// Send to channel (requires admin)
await sock.sendMessage('120363424098891946@newsletter', {
  text: 'Channel update message',
})
```

---

## 👥 Group management

```js
// Add participants
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'add')

// Remove participants
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'remove')

// Promote to admin
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'promote')

// Demote from admin
await sock.groupParticipantsUpdate(jid, ['521234567890@s.whatsapp.net'], 'demote')

// Get group metadata
const meta = await sock.groupMetadata(jid)
console.log(meta.subject)       // group name
console.log(meta.participants)  // participant list
console.log(meta.desc)          // description

// Change group name
await sock.groupUpdateSubject(jid, 'New Group Name')

// Change description
await sock.groupUpdateDescription(jid, 'New description')

// Get invite link
const code = await sock.groupInviteCode(jid)
console.log(`https://chat.whatsapp.com/${code}`)

// Revoke invite link
await sock.groupRevokeInvite(jid)

// Only admins can send (announcement mode)
await sock.groupSettingUpdate(jid, 'announcement')
await sock.groupSettingUpdate(jid, 'not_announcement')
```

---

## ⚠️ Compatibility notes

### Button types

| Type | Field | Works in | Notes |
|---|---|---|---|
| Simple buttons | `buttons[]` | Personal & Groups | Most compatible |
| Interactive flow | `interactiveButtons[]` | Personal & Groups | Requires megatoru |
| List (single_select) | `interactiveButtons[{name:'single_select'}]` | Personal & Groups | Requires megatoru |
| Legacy `buttonsMessage` | — | ❌ | Removed in Baileys v7+ |
| Legacy `listMessage` | — | ❌ | Removed in Baileys v7+ |

### `viewOnce` field

Adding `viewOnce: true` to button messages prevents them from being forwarded and removes them from chat after interaction on some WhatsApp versions. Use it for menus and sensitive content.

### `headerType` values

| Value | Header type |
|---|---|
| `1` | Text only |
| `2` | Document |
| `3` | Image (empty) |
| `4` | Image |
| `5` | Video |
| `6` | Location |

---

<div align="center">

Missing something? Open an [issue](https://github.com/anse-ux/megatoru/issues) or submit a PR.

**[← README](README.md)** · **[base-toru →](https://github.com/anse-ux/base-toru)**

[![anse-ux](https://img.shields.io/badge/developer-anse--ux-58a6ff?style=flat-square&logo=github&logoColor=white)](https://github.com/anse-ux)

</div>
