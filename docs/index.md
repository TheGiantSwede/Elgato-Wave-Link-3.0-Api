---
layout: default
title: Wave Link 3.0 Documentation
---

# Wave Link 3.0 API Documentation

Complete guide for integrating with Elgato Wave Link 3.0

## Overview

Wave Link is Elgato's professional audio routing software. Control individual application volumes, create custom audio mixes, and automate audio routing via API.

## What You Can Do

✅ **Control Audio Levels** - Set volume for any app in any mix
✅ **Mute/Unmute** - Toggle mute state per app
✅ **Custom Mixes** - Route apps to different destinations
✅ **Automate** - Build scripts and integrations
✅ **Stream Integration** - Hide/show apps in your stream

## Getting Started

### 1. Check Connection
```bash
netstat -ano | findstr :1884
```
Should show Wave Link listening on port 1884

### 2. Read the API Guide
[View full API documentation →](wave-link-api.md)

### 3. Quick Example

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
```

## Common IDs

### Channels (Sources)
- **Game** → `PCM_OUT_00_V_08_SD5`
- **Music** → `PCM_OUT_00_V_06_SD4`
- **Voice** → `PCM_OUT_00_V_02_SD2`
- **Browser** → `PCM_OUT_00_V_04_SD3`

### Mixes (Destinations)
- **Personal Mix** → `PCM_IN_01_V_00_SD1` (what you hear)
- **Stream Mix** → `PCM_IN_01_V_04_SD3` (what viewers see)
- **Chat Mix** → `PCM_IN_01_V_02_SD2`
- **Record Mix** → `PCM_IN_01_V_06_SD4`

## Popular Use Cases

### Stream Setup
Hide Discord from stream, show Game + Music

```javascript
// Set Game 100%, Music 50%, Voice Muted in Stream Mix
```
[See full example →](wave-link-api.md#stream-setup-example)

### Meeting Mode
Auto-adjust volumes when in calls

```javascript
// Mute music, max out Discord
```
[See full example →](wave-link-api.md#set-volume-function)

### Automation Scripts
Create profiles and presets

[See examples →](wave-link-api.md#common-patterns)

## Documentation

- **[API Reference](wave-link-api.md)** - Complete method documentation
- **[JavaScript Examples](wave-link-api.md#javascript)** - Code samples
- **[React Integration](wave-link-api.md#react-hook)** - React hook implementation
- **[Troubleshooting](wave-link-api.md#troubleshooting)** - Common issues & fixes

## Key Features

**JSON-RPC 2.0 Protocol**
```
Protocol: WebSocket
URL: ws://127.0.0.1:1884
Port: 1884
```

**Simple API**
- 4 main methods
- Standard JSON requests
- Real-time control

**Volume Format**
- Range: 0.0 - 1.0
- Example: 0.5 = 50%

## Installation

```bash
npm install ws
```

## Quick Test

```javascript
// test-connection.js
const net = require('net');

const socket = net.createConnection(1884, '127.0.0.1');
socket.on('connect', () => {
  console.log('✅ Connected to Wave Link');
  socket.destroy();
});
socket.on('error', () => {
  console.log('❌ Cannot reach Wave Link');
});
```

Run:
```bash
node test-connection.js
```

## Need Help?

1. **Check Wave Link is running** - Port 1884 should be listening
2. **Read the [API Guide](wave-link-api.md)** - Full documentation with examples
3. **See [Troubleshooting](wave-link-api.md#troubleshooting)** - Common issues

## Resources

- [Elgato Wave Link](https://www.elgato.com/en/wave-link)
- [API Guide](wave-link-api.md)
- [GitHub Issues](https://github.com)

---

**Wave Link Version:** 3.0.0.1635
**Last Updated:** November 2025
**Status:** ✅ Production Ready
