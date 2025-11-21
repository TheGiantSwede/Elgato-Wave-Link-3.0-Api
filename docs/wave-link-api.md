---
layout: default
title: Wave Link 3.0 API Guide
description: Simple reference for Elgato Wave Link 3.0 integration
---

# Wave Link 3.0 API Guide

Simple reference for integrating with Elgato Wave Link 3.0

## Connection

```
Protocol: WebSocket (JSON-RPC 2.0)
URL:      ws://127.0.0.1:1884
Port:     1884
```

## Methods

| Method | Purpose |
|--------|---------|
| `getChannels` | List all audio sources |
| `getMixes` | List all mixes |
| `getAppInfo` | Get Wave Link version |
| `setChannel` | Set volume or mute for channel in mix |

## Channel IDs (Sources)

Sources that send audio:

```
Game:     PCM_OUT_00_V_08_SD5
Browser:  PCM_OUT_00_V_04_SD3
Music:    PCM_OUT_00_V_06_SD4
Voice:    PCM_OUT_00_V_02_SD2
Mic:      PCM_IN_01_C_00_SD1
```

## Mix IDs (Destinations)

Where audio gets routed to:

```
Personal Mix:  PCM_IN_01_V_00_SD1  (what you hear)
Stream Mix:    PCM_IN_01_V_04_SD3  (what viewers see)
Chat Mix:      PCM_IN_01_V_02_SD2  (chat only)
Record Mix:    PCM_IN_01_V_06_SD4  (recording output)
```

## Request Format

All requests use JSON-RPC 2.0:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "methodName",
  "params": {}
}
```

## Examples

### Get All Channels

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "getChannels",
  "params": {}
}
```

Response:
```json
{
  "id": 1,
  "result": [
    {
      "id": "PCM_OUT_00_V_08_SD5",
      "name": "Game",
      "level": 0.75,
      "isMuted": false,
      "mixes": [
        {
          "id": "PCM_IN_01_V_00_SD1",
          "level": 0.75,
          "isMuted": false
        }
      ]
    }
  ]
}
```

### Set Volume

Set Music to 50% in Personal Mix:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "setChannel",
  "params": {
    "id": "PCM_OUT_00_V_06_SD4",
    "mixes": [
      {
        "id": "PCM_IN_01_V_00_SD1",
        "level": 0.5
      }
    ]
  }
}
```

**Important:** Level is `0.0-1.0` format, not 0-100
- `0.0` = 0%
- `0.5` = 50%
- `1.0` = 100%

### Mute Channel

Mute Game in Stream Mix:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "setChannel",
  "params": {
    "id": "PCM_OUT_00_V_08_SD5",
    "mixes": [
      {
        "id": "PCM_IN_01_V_04_SD3",
        "isMuted": true
      }
    ]
  }
}
```

## JavaScript

### Basic Example

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('ws://127.0.0.1:1884');

ws.on('open', () => {
  // Get all channels
  ws.send(JSON.stringify({
    id: 1,
    jsonrpc: '2.0',
    method: 'getChannels',
    params: {}
  }));
});

ws.on('message', (data) => {
  const response = JSON.parse(data);
  console.log(response.result);
  ws.close();
});

ws.on('error', (err) => {
  console.error('Connection error:', err.message);
});
```

### Set Volume Function

```javascript
function setVolume(channelId, mixId, volumePercent) {
  const ws = new WebSocket('ws://127.0.0.1:1884');

  ws.on('open', () => {
    ws.send(JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: channelId,
        mixes: [{
          id: mixId,
          level: volumePercent / 100
        }]
      }
    }));
  });

  ws.on('message', () => ws.close());
  ws.on('error', (err) => console.error(err));
}

// Usage:
setVolume('PCM_OUT_00_V_06_SD4', 'PCM_IN_01_V_00_SD1', 50);
```

### Stream Setup Example

Configure streaming mix (Game + Music visible, Discord hidden):

```javascript
const ws = new WebSocket('ws://127.0.0.1:1884');
let messageId = 1;

const streamConfig = [
  {
    channel: 'PCM_OUT_00_V_08_SD5',  // Game
    mix: 'PCM_IN_01_V_04_SD3',       // Stream Mix
    volume: 100
  },
  {
    channel: 'PCM_OUT_00_V_06_SD4',  // Music
    mix: 'PCM_IN_01_V_04_SD3',       // Stream Mix
    volume: 50
  },
  {
    channel: 'PCM_OUT_00_V_02_SD2',  // Voice/Discord
    mix: 'PCM_IN_01_V_04_SD3',       // Stream Mix
    volume: 0,
    isMuted: true                    // Mute Discord from stream
  }
];

ws.on('open', () => {
  streamConfig.forEach(config => {
    ws.send(JSON.stringify({
      id: messageId++,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: config.channel,
        mixes: [{
          id: config.mix,
          level: config.volume / 100,
          isMuted: config.isMuted || false
        }]
      }
    }));
  });

  setTimeout(() => ws.close(), 500);
});

ws.on('error', (err) => console.error(err));
```

## React Hook

```javascript
import { useState, useRef } from 'react';

export function useWaveLink() {
  const [connected, setConnected] = useState(false);
  const [error, setError] = useState(null);
  const wsRef = useRef(null);
  let messageId = 0;

  const connect = async () => {
    return new Promise((resolve, reject) => {
      try {
        wsRef.current = new WebSocket('ws://127.0.0.1:1884');

        wsRef.current.onopen = () => {
          setConnected(true);
          setError(null);
          resolve();
        };

        wsRef.current.onerror = (err) => {
          setError('Cannot connect to Wave Link');
          reject(err);
        };
      } catch (err) {
        reject(err);
      }
    });
  };

  const setVolume = (channelId, mixId, volumePercent) => {
    if (!wsRef.current || wsRef.current.readyState !== WebSocket.OPEN) {
      return false;
    }

    wsRef.current.send(JSON.stringify({
      id: ++messageId,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: channelId,
        mixes: [{
          id: mixId,
          level: volumePercent / 100
        }]
      }
    }));

    return true;
  };

  const disconnect = () => {
    if (wsRef.current) {
      wsRef.current.close();
      setConnected(false);
    }
  };

  return {
    connected,
    error,
    connect,
    setVolume,
    disconnect
  };
}
```

## Troubleshooting

### Connection Refused

**Problem:** Cannot connect to `ws://127.0.0.1:1884`

**Solution:**
1. Verify Wave Link is running
2. Check port 1884 is open:
   ```bash
   netstat -ano | findstr :1884
   ```
3. Expected output: `LISTENING 127.0.0.1:1884`

### Invalid Method Error

**Problem:** `Method not found` error

**Solution:** Use correct method name `setChannel`, not `setVolume` or `updateChannel`

### Volume Not Changing

**Problem:** Volume value sent but nothing happens

**Solutions:**
- Check level format: must be `0.0-1.0`, not `0-100`
  - ✅ Correct: `"level": 0.5`
  - ❌ Wrong: `"level": 50`
- Verify channel ID is correct
- Confirm mix ID matches channel's available mixes

### Parameter Error

**Problem:** Invalid parameters error

**Solutions:**
- Use `isMuted` not `mute` or `muted`
  - ✅ Correct: `"isMuted": true`
  - ❌ Wrong: `"mute": true`
- Ensure params structure matches example
- Check all required fields are present

## Testing

### Test Connection Script

```javascript
const net = require('net');

const socket = net.createConnection(1884, '127.0.0.1');

socket.on('connect', () => {
  console.log('✅ Wave Link is running');
  socket.destroy();
});

socket.on('error', (err) => {
  console.log('❌ Cannot reach Wave Link on port 1884');
  console.log('Error:', err.message);
});

socket.setTimeout(2000, () => {
  console.log('❌ Timeout - no response');
  socket.destroy();
});
```

Run:
```bash
node test.js
```

## Install Dependencies

```bash
npm install ws
```

## Common Patterns

### Batch Set Multiple Volumes

```javascript
async function applyMixPreset(preset) {
  const ws = new WebSocket('ws://127.0.0.1:1884');
  let id = 1;

  ws.on('open', () => {
    preset.forEach(item => {
      ws.send(JSON.stringify({
        id: id++,
        jsonrpc: '2.0',
        method: 'setChannel',
        params: {
          id: item.channel,
          mixes: [{
            id: item.mix,
            level: item.volume / 100
          }]
        }
      }));
    });

    setTimeout(() => ws.close(), 500);
  });
}

// Usage:
applyMixPreset([
  { channel: 'PCM_OUT_00_V_08_SD5', mix: 'PCM_IN_01_V_00_SD1', volume: 80 },
  { channel: 'PCM_OUT_00_V_06_SD4', mix: 'PCM_IN_01_V_00_SD1', volume: 60 }
]);
```

### Persistent Connection

```javascript
class WaveLinkClient {
  constructor() {
    this.ws = null;
    this.messageId = 0;
  }

  async connect() {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket('ws://127.0.0.1:1884');
      this.ws.onopen = () => resolve();
      this.ws.onerror = reject;
    });
  }

  setVolume(channelId, mixId, volumePercent) {
    if (!this.ws || this.ws.readyState !== WebSocket.OPEN) {
      throw new Error('Not connected');
    }

    this.ws.send(JSON.stringify({
      id: ++this.messageId,
      jsonrpc: '2.0',
      method: 'setChannel',
      params: {
        id: channelId,
        mixes: [{
          id: mixId,
          level: volumePercent / 100
        }]
      }
    }));
  }

  disconnect() {
    if (this.ws) this.ws.close();
  }
}

// Usage:
const client = new WaveLinkClient();
await client.connect();
client.setVolume('PCM_OUT_00_V_06_SD4', 'PCM_IN_01_V_00_SD1', 50);
client.disconnect();
```

## Links

- [Wave Link Official](https://www.elgato.com/en/wave-link)
- [GitHub Repository](https://github.com)
- [Issues & Questions](https://github.com/issues)

---

**Last Updated:** November 2025
**Wave Link Version:** 3.0.0.1635
**Protocol:** JSON-RPC 2.0 over WebSocket
**Status:** Production Ready ✅
