# Electron.js with React.js (Vite)

This repository demonstrates how to build a desktop application using **Electron.js** with **React.js** and **Vite**.

## 📌 Prerequisites
- Node.js (Latest LTS recommended)
- npm or yarn
- Basic knowledge of React.js and Electron.js

## 📂 Project Setup

### 1️⃣ Initialize a React Project with Vite
```sh
npm create vite@latest electron-react-app --template react
cd electron-react-app
npm install
```

### 2️⃣ Install Electron
```sh
npm install electron electron-builder --save-dev
```

### 3️⃣ Create `electron/main.js`
Create an **electron/main.js** file to manage the Electron main process.

```js
const { app, BrowserWindow } = require('electron');
const path = require('path');

let mainWindow;

app.whenReady().then(() => {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });
  mainWindow.loadURL(`file://${path.join(__dirname, '../dist/index.html')}`);
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

### 4️⃣ Modify `package.json`
Add an Electron start script inside `package.json`:

```json
"main": "electron/main.js",
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "electron": "electron .",
  "electron-dev": "vite build && electron ."
}
```

### 5️⃣ Run the Application
Start React:
```sh
npm run dev
```
Start Electron:
```sh
npm run electron-dev
```

## 🚀 Build for Production
To package the app as an executable:
```sh
npm run build && npm run electron
```

## 🎯 Features
- Desktop App using React.js and Vite
- Fast Development with Vite
- Simple Electron.js Integration
- Production Build Packaging

## 📜 License
This project is licensed under the MIT License.

