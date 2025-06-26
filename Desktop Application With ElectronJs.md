# ⚡ Electron + Vite + React from Scratch

Create a modern desktop application using **Electron**, **Vite**, and **React**, starting from zero using `npm create vite`.

---

## 🚀 Project Setup

### 1. Create a Vite + React Project

```bash
npm create vite@latest electron-vite-app
```

When prompted:
- **Project name**: `electron-vite-app`
- **Framework**: `React`
- **Variant**: `JavaScript` (or `TypeScript` if you prefer)

```bash
cd electron-vite-app
npm install
```

---

### 2. Install Electron & Tools

```bash
npm install --save-dev electron concurrently wait-on electron-builder
```

---

### 3. Setup Project Structure

Create a folder for Electron:

```bash
mkdir electron
```

#### Create `electron/main.js`

```js
// electron/main.js
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
    },
  });

  const isDev = !app.isPackaged;
  if (isDev) {
    win.loadURL('http://localhost:5173');
  } else {
    win.loadFile(path.join(__dirname, '../dist/index.html'));
  }
}

app.whenReady().then(createWindow);
```

#### Create `electron/preload.js`

```js
// electron/preload.js
const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('api', {
  ping: () => 'pong',
});
```

---

## ⚙️ Configuration

### 4. Update `vite.config.js`

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  base: './', // For Electron compatibility
  plugins: [react()],
});
```

---

### 5. Update `package.json`

Add scripts and build config:

```jsonc
{
  "main": "electron/main.js",
  "scripts": {
    "dev": "concurrently \"vite\" \"wait-on http://localhost:5173 && electron .\"",
    "build": "vite build",
    "make:win": "npm run build && electron-builder --win"
  },
  "build": {
    "appId": "com.example.electronvite",
    "productName": "ElectronViteApp",
    "files": [
      "dist/",
      "electron/"
    ],
    "directories": {
      "output": "dist"
    },
    "win": {
      "target": "nsis"
    }
  }
}
```

---

## 🧪 Run and Test

### Start Development

```bash
npm run dev
```

### Build Production

```bash
npm run build
```

### Package for Windows

```bash
npm run make:win
```

---

## 🧰 Common Issues

### `ENOENT: preload.js not found`

Ensure you reference preload using:

```js
preload: path.join(__dirname, 'preload.js')
```

### `ERR_FILE_NOT_FOUND` for JS/CSS

Fix with:

```js
base: './'
```

in `vite.config.js`.

### `EBUSY` during build

- Close any running `.exe`
- Exit Explorer windows in `dist/`
- Run terminal as Administrator

---

## 📤 Share with Friends

- Installer: Share `dist/ElectronViteApp Setup.exe`
- Portable: Share `dist/win-unpacked/` and tell them to run `YourApp.exe`

If SmartScreen warns, tell them to click **"More info > Run anyway"**.

---

## 📝 License

[MIT](LICENSE)
