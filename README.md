# App Volume Mixer

A modern, standalone desktop volume mixer application with hardware control support via serial communication. Control individual application volumes using physical potentiometers, encoders, and buttons.

**Now available as a native desktop application using Electron!**

## Features

- **Volume Control**: Individual volume sliders for different applications (Spotify, Discord, Chrome, Steam, etc.)
- **Hardware Integration**: Connect physical controls via Web Serial API
- **Multiple Control Types**:
  - **Potentiometers**: Analog volume control
  - **Encoders**: Rotary encoders with button press for dual functionality
  - **Buttons**: Quick mute/unmute toggles
- **Visual Feedback**: Real-time audio visualizer bars on volume sliders
- **Customizable**: Configure which application each control manages
- **Serial Channels**: Support for multiple serial channels (CH0-CH4)

## Prerequisites

- Node.js (v16 or higher) - Only needed for development
- npm or yarn - Only needed for development

## Installation

1. Install dependencies:
```bash
npm install
```

## Running the Application

### Option 1: Standalone Desktop App (Recommended)

Run the Electron app in development mode:
```bash
npm run electron
```
This will launch a native desktop window with the app running.

### Option 2: Web Browser Mode

Start the development server with hot reload:
```bash
npm run dev
```
Then open your browser to `http://localhost:3000`

## Building Standalone Executables

Build the app as a standalone executable that doesn't require a browser:

### Windows
```bash
npm run electron:build:win
```
Creates an installer (`App-Volume-Mixer-Setup-1.0.0.exe`) and portable version in the `release/` folder.

### macOS
```bash
npm run electron:build:mac
```
Creates a `.dmg` installer in the `release/` folder.

### Linux
```bash
npm run electron:build:linux
```
Creates an `.AppImage` and `.deb` package in the `release/` folder.

### Build for All Platforms
```bash
npm run electron:build
```

## Usage

### Web Interface

1. **Add Controls**: Click "Add Control" to create a new volume control
2. **Settings**: Click the "Settings" button to configure each control:
   - Select the application to control
   - Choose control type (Potentiometer, Encoder, or Button)
   - Assign a serial channel (CH0-CH4)
   - For encoders, configure both rotation and button press actions
3. **Remove Controls**: Hover over a control and click the trash icon to remove it

### Hardware Connection

1. Click the "Connect" button
2. Select your serial device from the browser dialog
3. The button will turn green when connected

### Serial Protocol

The application expects serial data in the format: `CHANNEL:VALUE\n`

- **CHANNEL**: One of CH0, CH1, CH2, CH3, CH4
- **VALUE**: Integer from 20-1020 (mapped to 0-100%)

#### Examples:
```
CH0:512\n    # Set channel 0 to ~50%
CH1:1020\n   # Set channel 1 to 100%
CH2:20\n     # Set channel 2 to 0%
CH3:1\n      # Trigger button on channel 3
```

#### Control Type Behaviors:
- **Potentiometers**: VALUE is mapped to volume (20-1020 → 0-100%)
- **Encoders**: VALUE controls volume, button press on configured channel toggles mute
- **Buttons**: Any VALUE ≥ 1 toggles the mute state

### Arduino Example

```cpp
void setup() {
  Serial.begin(9600);
  pinMode(A0, INPUT);
}

void loop() {
  int potValue = analogRead(A0);
  int mappedValue = map(potValue, 0, 1023, 20, 1020);

  Serial.print("CH0:");
  Serial.println(mappedValue);

  delay(50);
}
```

## Configuration

### Available Applications
- Spotify
- Discord
- Slack
- System Sounds
- Google Chrome
- YouTube
- Steam

### Control Types

**Potentiometer**
- Simple analog volume control
- Maps serial value directly to volume

**Encoder**
- Rotation controls volume of one application
- Button press (separate channel) mutes a different application
- Ideal for dual-function controls

**Button**
- Toggle mute/unmute
- Triggers on any serial value ≥ 1

## Compatibility

### Desktop App (Electron)
- Windows 10/11
- macOS 10.13+
- Linux (most distributions)
- Serial port support built-in

### Web Browser Mode
Requires the Web Serial API, which is supported in:
- Google Chrome (89+)
- Microsoft Edge (89+)
- Opera (76+)

**Note**: Firefox and Safari do not currently support the Web Serial API.

## Technologies Used

- **Electron** - Desktop application framework
- **React 18** - UI framework
- **TypeScript** - Type safety
- **Vite** - Build tool
- **Tailwind CSS** - Styling
- **Lucide React** - Icons
- **Web Serial API** - Hardware communication

## Project Structure

```
DeejTGS1/
├── src/
│   ├── main.tsx          # Application entry point
│   ├── volume-ui.jsx     # Main volume mixer component
│   └── index.css         # Global styles with Tailwind
├── electron/
│   ├── main.js           # Electron main process
│   └── preload.js        # Electron preload script
├── index.html            # HTML template
├── package.json          # Dependencies and scripts
├── vite.config.ts        # Vite configuration
├── tailwind.config.js    # Tailwind CSS configuration
└── postcss.config.js     # PostCSS configuration
```

## License

MIT
