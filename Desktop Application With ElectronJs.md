# MERN Stack with Electron and Vite

This guide will help you set up a **MERN (MongoDB, Express, React, Node.js) stack** with **Electron** for building a desktop application.

## **1. Setup the Backend (Express & MongoDB)**

### **1.1 Initialize a Node.js Project**
```sh
mkdir mern-electron-app && cd mern-electron-app
mkdir backend && cd backend
npm init -y
```

### **1.2 Install Dependencies**
```sh
npm install express mongoose cors dotenv
npm install nodemon --save-dev
```

### **1.3 Setup Express Server**
Create a new file `backend/server.js` and add the following:

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB connected"))
.catch(err => console.error(err));

app.get("/", (req, res) => {
  res.send("Hello from the backend");
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### **1.4 Create a `.env` File**
Create a `.env` file in the backend directory:
```
MONGO_URI=your_mongodb_connection_string
```

### **1.5 Start the Backend Server**
Modify `package.json` scripts:
```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}
```
Run the backend server:
```sh
npm run dev
```

---
## **2. Setup the Frontend (React + Vite + Electron)**

### **2.1 Create a React Project with Vite**
```sh
cd ..
mkdir frontend && cd frontend
npm create vite@latest . --template react
npm install
```

### **2.2 Install Electron**
```sh
npm install --save-dev electron electron-builder concurrently wait-on
```

### **2.3 Configure Electron**
Create `frontend/public/electron.js`:

```javascript
const { app, BrowserWindow } = require("electron");
const path = require("path");

let mainWindow;
app.whenReady().then(() => {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true,
    },
  });

  mainWindow.loadURL(
    process.env.NODE_ENV === "development"
      ? "http://localhost:5173"
      : `file://${path.join(__dirname, "../dist/index.html")}`
  );
});
```

### **2.4 Modify `package.json`**
Modify `frontend/package.json`:
```json
"main": "public/electron.js",
"scripts": {
  "dev": "concurrently \"vite\" \"wait-on http://localhost:5173 && electron public/electron.js\"",
  "build": "vite build && electron-builder",
  "electron": "electron public/electron.js"
},
"build": {
  "appId": "com.mern.electron.app",
  "productName": "MERN Electron App",
  "directories": {
    "output": "dist"
  },
  "files": [
    "dist/**/*",
    "public/electron.js"
  ],
  "win": {
    "target": "nsis",
    "icon": "public/icon.png"
  }
}
```

### **2.5 Start the Frontend & Electron**
```sh
npm run dev
```

---
## **3. Connect Frontend with Backend**
### **3.1 Install Axios in Frontend**
```sh
npm install axios
```

### **3.2 Create an API Service**
Create `frontend/src/api.js`:
```javascript
import axios from "axios";

const API = axios.create({ baseURL: "http://localhost:5000" });

export const fetchData = async () => {
  const response = await API.get("/");
  return response.data;
};
```

### **3.3 Fetch Data in React Component**
Modify `frontend/src/App.jsx`:
```javascript
import { useEffect, useState } from "react";
import { fetchData } from "./api";

function App() {
  const [data, setData] = useState("");

  useEffect(() => {
    fetchData().then(response => setData(response));
  }, []);

  return (
    <div>
      <h1>Electron + React + Express</h1>
      <p>Backend says: {data}</p>
    </div>
  );
}

export default App;
```

### **3.4 Start Both Backend and Frontend**
In separate terminals:
```sh
cd backend && npm run dev
```
```sh
cd frontend && npm run dev
```

---
## **4. Build and Package the App**
```sh
cd frontend
npm run build
npm run electron
```
To generate an installer:
```sh
npm run build
```

---
## **Conclusion**
You now have a full **MERN stack desktop application** using **Electron** and **Vite**! 🚀

