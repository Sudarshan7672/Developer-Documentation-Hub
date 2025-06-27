# ⚡ Electron + Vite + React from Scratch

Create a modern desktop application using **Electron**, **Vite**, and **React**, starting from zero using `npm create vite`.

---

## 🚀 Project Setup
---
Note - Don't apply tailwind until all steps are completed successfully
---

### 1. Create a Vite + React Project

```bash
npm create vite electron-vite-app
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

## 📁 Project Structure

```
electron-vite-app/
├── build/                  # Custom NSIS scripts
│   └── installer.nsh
├── electron/               # Electron main and preload
│   ├── main.cjs
│   └── preload.js
├── public/
├── src/
├── dist/                   # Built output
├── package.json
└── vite.config.js
```

---

## 🧱 Setup Electron

### Create `electron/main.cjs`

```js
// electron/main.cjs
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

---

### Create `electron/preload.js`

```js
// electron/preload.js
const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('api', {
  ping: () => 'pong',
});
```

---

## ⚙️ Configuration

### `vite.config.js`

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  base: './',
  plugins: [react()],
});
```

---

### Updated `package.json`

```json
{
  "name": "electron-vite-app",
  "version": "1.0.0",
  "type": "module",
  "main": "electron/main.cjs",
  "scripts": {
    "dev": "concurrently \"vite\" \"wait-on http://localhost:5173 && electron .\"",
    "build": "vite build",
    "make:win": "npm run build && electron-builder --win"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "concurrently": "^8.2.2",
    "electron": "^29.0.0",
    "electron-builder": "^24.13.1",
    "vite": "^6.3.5",
    "@vitejs/plugin-react": "^4.0.3",
    "wait-on": "^7.0.1"
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
    },
    "nsis": {
    "oneClick": false,
    "perMachine": false,
    "allowToChangeInstallationDirectory": true,
    "include": "build/installer.nsh"
  }
  }
}
```

---

## 🔐 Add Product Key Check (NSIS)

### 1. Create `build/installer.nsh`

```nsh
!include "MUI2.nsh"
!include "LogicLib.nsh"
!include "nsDialogs.nsh"

Var InputField
Var Label

# 🧩 Attach both page & leave function
Page custom ProductKeyPage ProductKeyPageLeave

Function ProductKeyPage
  nsDialogs::Create 1018
  Pop $0
  ${If} $0 == error
    MessageBox MB_ICONSTOP "Failed to create dialog"
    Abort
  ${EndIf}

  ${NSD_CreateLabel} 0u 0u 100% 12u "Enter your product key to continue installation:"
  Pop $Label

  ${NSD_CreateText} 0u 14u 100% 12u ""
  Pop $InputField

  nsDialogs::Show
FunctionEnd

Function ProductKeyPageLeave
  ${NSD_GetText} $InputField $R0
  ${If} $R0 != "shubhamshravani"
    MessageBox MB_ICONSTOP "Invalid product key. Setup will now exit."
    Abort
  ${EndIf}
FunctionEnd
```

> 💡 You can store valid keys in an encrypted file or check online with more advanced logic.

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

### Create Windows Installer with Product Key Check

```bash
npm run make:win
```

---

## 🧰 Common Issues

### `ENOENT: preload.js not found`

Make sure `preload` is referenced correctly:

```js
preload: path.join(__dirname, 'preload.js')
```

### `ERR_FILE_NOT_FOUND` for JS/CSS

Ensure `vite.config.js` contains:

```js
base: './'
```

### `EBUSY` during build

- Close any running `.exe`
- Close File Explorer in `dist/`
- Run your terminal as **Administrator**

---

## 📤 Share with Friends

- Share `dist/ElectronViteApp Setup.exe` (installer)
- Or zip and share `dist/win-unpacked/` for portable use

If Windows SmartScreen shows a warning, advise users to click:
**More info → Run anyway**

---

## 📝 License

[MIT](LICENSE)
