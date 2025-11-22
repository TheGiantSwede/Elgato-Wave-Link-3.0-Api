# Elgato Wave Link 3.0 API

Simple, practical reference for integrating with **Elgato Wave Link 3.0** via WebSocket API.

Control audio routing, volume levels, and muting for individual applications programmatically.

## Quick Start

### 1. Check Wave Link is Running

```bash
netstat -ano | findstr :1884
# Expected: LISTENING 127.0.0.1:1884
```

### 2. Install Dependencies

```bash
npm install ws
```

### 3. First Command

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('ws://127.0.0.1:1884');

ws.on('open', () => {
  // Set a channel to 50% in a mix
  // Replace channelId and mixId with your actual IDs from getChannels()
  ws.send(JSON.stringify({
    id: 1,
    jsonrpc: '2.0',
    method: 'setChannel',
    params: {
      id: 'YOUR_CHANNEL_ID',     // e.g., "PCM_OUT_00_V_06_SD4"
      mixes: [{
        id: 'YOUR_MIX_ID',         // e.g., "PCM_IN_01_V_00_SD1"
        level: 0.5                 // 50%
      }]
    }
  }));
});

ws.on('message', () => {
  console.log('✅ Volume changed!');
  ws.close();
});
```

## Get Your Channel & Mix IDs

IDs are different on every PC. Discover them dynamically:

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('ws://127.0.0.1:1884');

ws.on('open', () => {
  ws.send(JSON.stringify({
    id: 1,
    jsonrpc: '2.0',
    method: 'getChannels',
    params: {}
  }));
});

ws.on('message', (data) => {
  const response = JSON.parse(data);

  console.log('Channels:');
  response.result.forEach(ch => {
    console.log(`  ${ch.name}: ${ch.id}`);
  });

  ws.close();
});
```

Then get mixes:

```javascript
ws.send(JSON.stringify({
  id: 2,
  jsonrpc: '2.0',
  method: 'getMixes',
  params: {}
}));
```

## API Methods

### Get Channels
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "getChannels",
  "params": {}
}
```

### Set Volume
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "setChannel",
  "params": {
    "id": "YOUR_CHANNEL_ID",
    "mixes": [{
      "id": "YOUR_MIX_ID",
      "level": 0.5
    }]
  }
}
```

### Mute Channel
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "setChannel",
  "params": {
    "id": "YOUR_CHANNEL_ID",
    "mixes": [{
      "id": "YOUR_MIX_ID",
      "isMuted": true
    }]
  }
}
```

## Common Examples

### Batch Set Multiple Channels

```javascript
const ws = new WebSocket('ws://127.0.0.1:1884');
let id = 1;

// Replace with your actual IDs from getChannels() and getMixes()
const config = [
  { channel: 'YOUR_GAME_ID', mix: 'YOUR_STREAM_MIX_ID', volume: 100 },
  { channel: 'YOUR_MUSIC_ID', mix: 'YOUR_STREAM_MIX_ID', volume: 50 },
  { channel: 'YOUR_VOICE_ID', mix: 'YOUR_STREAM_MIX_ID', mute: true }
];

ws.on('open', () => {
  config.forEach(c => {
    ws.send(JSON.stringify({
      id: id++,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: c.channel,
        mixes: [{
          id: c.mix,
          level: c.volume ? c.volume / 100 : 0,
          isMuted: c.mute || false
        }]
      }
    }));
  });
  setTimeout(() => ws.close(), 500);
});
```

### Dynamic Meeting Mode

```javascript
function setMeetingMode(isActive, voiceChannelId, musicChannelId, personalMixId) {
  const ws = new WebSocket('ws://127.0.0.1:1884');
  let id = 1;

  ws.on('open', () => {
    const levels = isActive
      ? { voice: 100, music: 0 }      // Meeting: max voice, mute music
      : { voice: 30, music: 100 };    // Normal: quiet voice, full music

    const channels = [
      { id: voiceChannelId, level: levels.voice },
      { id: musicChannelId, level: levels.music }
    ];

    channels.forEach(c => {
      ws.send(JSON.stringify({
        id: id++,
        jsonrpc: '2.0',
        method: 'setChannel',
        params: {
          id: c.id,
          mixes: [{
            id: personalMixId,
            level: c.level / 100
          }]
        }
      }));
    });
    setTimeout(() => ws.close(), 500);
  });
}

// Usage:
// setMeetingMode(true, voiceId, musicId, personalMixId);
```

## React Hook

```javascript
import { useState, useRef } from 'react';

export function useWaveLink() {
  const [connected, setConnected] = useState(false);
  const [channels, setChannels] = useState([]);
  const [mixes, setMixes] = useState([]);
  const wsRef = useRef(null);
  let msgId = 0;

  const connect = () => {
    return new Promise((resolve) => {
      wsRef.current = new WebSocket('ws://127.0.0.1:1884');
      wsRef.current.onopen = () => {
        setConnected(true);
        resolve();
      };
    });
  };

  const fetchChannels = () => {
    if (!wsRef.current) return;

    wsRef.current.send(JSON.stringify({
      id: ++msgId,
      jsonrpc: '2.0',
      method: 'getChannels',
      params: {}
    }));
  };

  const setVolume = (channelId, mixId, volume) => {
    if (!wsRef.current) return;

    wsRef.current.send(JSON.stringify({
      id: ++msgId,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: channelId,
        mixes: [{ id: mixId, level: volume / 100 }]
      }
    }));
  };

  return { connected, channels, mixes, connect, fetchChannels, setVolume };
}
```

## Important Notes

⚠️ **Volume Format**: Wave Link uses `0.0-1.0`, not `0-100`
- `0.0` = 0% (silent)
- `0.5` = 50%
- `1.0` = 100% (max)

⚠️ **Key Names**: Use `isMuted`, not `mute` or `muted`

⚠️ **Methods**: Use `setChannel`, not `setVolume` or `updateChannel`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **Connection refused** | Wave Link not running on port 1884 |
| **Invalid method** | Use `setChannel`, not `setVolume` |
| **Volume not changing** | Check level is 0.0-1.0, not 0-100 |
| **Permission denied** | Verify channel/mix IDs are correct |

## Full Documentation

See the complete API reference with more examples at:
- **[Full API Reference](docs/wave-link-api.md)**

## API Specification

**Protocol:** WebSocket (JSON-RPC 2.0)
**URL:** `ws://127.0.0.1:1884`
**Port:** `1884`
**Version:** 3.0.0.1635+

## Installation

```bash
npm install ws
```

## License

MIT
