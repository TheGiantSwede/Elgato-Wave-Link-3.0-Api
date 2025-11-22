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
  // Set Music to 50% in Personal Mix
  ws.send(JSON.stringify({
    id: 1,
    jsonrpc: '2.0',
    method: 'setChannel',
    params: {
      id: 'PCM_OUT_00_V_06_SD4',      // Music
      mixes: [{
        id: 'PCM_IN_01_V_00_SD1',      // Personal Mix
        level: 0.5                      // 50%
      }]
    }
  }));
});

ws.on('message', () => {
  console.log('✅ Volume changed!');
  ws.close();
});
```

## Channel IDs (Audio Sources)

| Name | ID |
|------|-----|
| **Game** | `PCM_OUT_00_V_08_SD5` |
| **Music** | `PCM_OUT_00_V_06_SD4` |
| **Voice/Discord** | `PCM_OUT_00_V_02_SD2` |
| **Browser** | `PCM_OUT_00_V_04_SD3` |
| **Microphone** | `PCM_IN_01_C_00_SD1` |

## Mix IDs (Audio Destinations)

| Name | ID | Purpose |
|------|-----|---------|
| **Personal Mix** | `PCM_IN_01_V_00_SD1` | What you hear (headphones) |
| **Stream Mix** | `PCM_IN_01_V_04_SD3` | What viewers see |
| **Chat Mix** | `PCM_IN_01_V_02_SD2` | Chat channel only |
| **Record Mix** | `PCM_IN_01_V_06_SD4` | Recording output |

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
    "id": "PCM_OUT_00_V_06_SD4",
    "mixes": [{
      "id": "PCM_IN_01_V_00_SD1",
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
    "id": "PCM_OUT_00_V_08_SD5",
    "mixes": [{
      "id": "PCM_IN_01_V_04_SD3",
      "isMuted": true
    }]
  }
}
```

## Common Examples

### Stream Setup (Game + Music, Hide Discord)

```javascript
const ws = new WebSocket('ws://127.0.0.1:1884');
let id = 1;

const config = [
  { ch: 'PCM_OUT_00_V_08_SD5', mix: 'PCM_IN_01_V_04_SD3', vol: 100 }, // Game
  { ch: 'PCM_OUT_00_V_06_SD4', mix: 'PCM_IN_01_V_04_SD3', vol: 50 },  // Music
  { ch: 'PCM_OUT_00_V_02_SD2', mix: 'PCM_IN_01_V_04_SD3', mute: true } // Discord
];

ws.on('open', () => {
  config.forEach(c => {
    ws.send(JSON.stringify({
      id: id++,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: c.ch,
        mixes: [{
          id: c.mix,
          level: c.vol ? c.vol / 100 : 0,
          isMuted: c.mute || false
        }]
      }
    }));
  });
  setTimeout(() => ws.close(), 500);
});
```

### Meeting Mode (Discord Loud, Music Quiet)

```javascript
function setMeetingMode(isActive) {
  const ws = new WebSocket('ws://127.0.0.1:1884');
  let id = 1;

  ws.on('open', () => {
    const channels = [
      { ch: 'PCM_OUT_00_V_02_SD2', vol: isActive ? 100 : 30 },  // Voice
      { ch: 'PCM_OUT_00_V_06_SD4', vol: isActive ? 0 : 100 }    // Music
    ];

    channels.forEach(c => {
      ws.send(JSON.stringify({
        id: id++,
        jsonrpc: '2.0',
        method: 'setChannel',
        params: {
          id: c.ch,
          mixes: [{
            id: 'PCM_IN_01_V_00_SD1',
            level: c.vol / 100
          }]
        }
      }));
    });
    setTimeout(() => ws.close(), 500);
  });
}

setMeetingMode(true);
```

## React Hook

```javascript
import { useState, useRef } from 'react';

export function useWaveLink() {
  const [connected, setConnected] = useState(false);
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

  return { connected, connect, setVolume };
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
