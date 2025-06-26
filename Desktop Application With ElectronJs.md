# 🚀 Electron Vite App

A modern desktop application built using **Electron**, **Vite**, and **React**. This project combines a fast frontend workflow with desktop app packaging, including `.exe` creation for Windows.

---

## 📁 Project Structure

```
electron-vite-app/
├── dist/                 # Production build output
├── electron/             # Electron main & preload scripts
│   ├── main.js
│   └── preload.js
├── public/               # Static public assets
├── src/                  # React source code
│   ├── main.jsx
│   └── App.jsx
├── vite.config.js        # Vite configuration
├── package.json          # Project metadata and scripts
└── README.md             # Documentation
```

---

## 🧱 Tech Stack

- ⚡ [Vite](https://vitejs.dev/) – Lightning-fast frontend bundler
- ⚛️ [React](https://reactjs.org/) – Component-based frontend
- 💻 [Electron](https://www.electronjs.org/) – Cross-platform desktop app runtime
- 📦 [electron-builder](https://www.electron.build/) – Installer and packaging tool

---

## 📦 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/electron-vite-app.git
cd electron-vite-app
```

### 2. Install Dependencies

```bash
npm install
```

---

## 🚧 Development

Run the app in development mode with hot reload:

```bash
npm run dev
```

- Starts the Vite dev server at `http://localhost:5173`
- Launches Electron and loads the frontend

---

## 🛠 Build & Package

### 1. Build the Frontend

```bash
npm run build
```

### 2. Package the App for Windows (.exe)

```bash
npm run make:win
```

This uses `electron-builder` to create:

- 📂 `dist/win-unpacked/` – Portable version (no install required)
- 📦 `dist/YourApp Setup.exe` – Installer file

> If you encounter permission issues or symbolic link errors, run your terminal as **Administrator**.

---

## 🧪 Test the Final App

- **Portable version**:  
  Run `dist/win-unpacked/YourApp.exe`

- **Installer version**:  
  Run the generated `.exe` installer

---

## 🔧 Configuration

### vite.config.js

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  base: './', // Important for Electron compatibility
  plugins: [react()],
});
```

### electron/main.js

```js
const { app, BrowserWindow } = require('electron');
const path = require('path');
const isDev = !app.isPackaged;

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
    },
  });

  if (isDev) {
    win.loadURL('http://localhost:5173');
  } else {
    win.loadFile(path.join(__dirname, '../dist/index.html'));
  }
}

app.whenReady().then(createWindow);
```

### package.json (build section)

```json
"main": "electron/main.js",
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
```

---

## 🧰 Common Issues & Fixes

### 🔸 `Cannot load preload.js`

Ensure `preload.js` exists and is referenced correctly:

```js
preload: path.join(__dirname, 'preload.js')
```

Also make sure `electron/` is included in `build.files`.

---

### 🔸 `GET file:///assets/index-XYZ.js not found`

Fix by setting:

```js
base: './'
```

in your `vite.config.js`.

---

### 🔸 `EBUSY: resource busy or locked`

Make sure the app is **not running** while building.
- Close any running `.exe`
- Exit File Explorer windows in the `dist/` directory
- Try again or restart terminal as Administrator

---

## 📤 Distributing the App

### Option 1: Share the Installer

- Zip the file: `dist/YourApp Setup 1.0.0.exe`
- Share via Google Drive, Dropbox, etc.

### Option 2: Share the Portable App

- Zip the folder: `dist/win-unpacked`
- Friend can run `YourApp.exe` directly (no install)

---

## 📌 Extras

Want to customize your build?

- Add icons
- Create a splash screen
- Add auto-launch at startup
- Add system tray integration
- Add auto-updates

Let me know, and I’ll help you set it up!

---

## 📝 License

This project is licensed under the [MIT License](LICENSE).
