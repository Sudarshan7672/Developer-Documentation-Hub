# Project Repository

This repository contains all the necessary files and documentation for setting up and deploying a project consisting of Frontend, Admin Panel, and Backend components. The following sections provide detailed instructions and information about the project structure and VPS hosting.

---

## Project Structure

### Main_Folder
The root directory contains three subfolders:

1. **frontend**
   - Contains the codebase for the user-facing frontend application.
   - Built using modern web technologies such as React.

2. **Admin**
   - Contains the codebase for the admin panel.
   - Built to manage application data and configurations.

3. **Backend**
   - Contains the codebase for the backend APIs.
   - Built with Node.js and Express.

Each subfolder has its own `package.json` file for managing dependencies.

---

## VPS Hosting Setup for Ubuntu OS

### Key Steps Overview:

1. **Connect to Your VPS and Install Dependencies.**
2. **Generate and Add SSH Keys to GitHub for Private Repository Access.**
3. **Clone the Repository from GitHub and Set Up the Environment.**
4. **Configure Nginx to Serve the Frontend and Admin Panels.**
5. **Test Nginx Configuration, Restart Nginx, and Enable It.**
6. **Set Up the Firewall and Enable Essential Ports.**
7. **Install SSL Certificates Using Let’s Encrypt.**

---

### Step-by-Step Guide

#### Step 1: Connect to Your VPS

1. SSH into your VPS:
   ```bash
   ssh root@your-vps-ip
   ```
2. Update your system and install dependencies:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y git nodejs npm nginx
   ```
3. Check Nginx installation:
   ```bash
   sudo systemctl status nginx
   ```

#### Step 2: Generate SSH Key and Add to GitHub

1. Generate a new SSH key pair:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
   ```
2. Add the SSH key to the SSH agent:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_rsa
   ```
3. Copy the public key and add it to GitHub:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

#### Step 3: Clone the Repository

Clone the `Main_Folder` repository:
```bash
git clone git@github.com:your-username/your-repository.git
cd your-repository
```

#### Step 4: Configure Nginx

Edit the Nginx configuration file:
```bash
sudo nano /etc/nginx/sites-available/default
```

Example configuration:
```nginx
server {  # Frontend Server
    listen 80;
    server_name main_domain www.main_domain;

    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

server {  # Backend API Server
    listen 80;
    server_name api.main_domain;

    location / {
        proxy_pass http://localhost:8080;
    }
}

server {  # Admin Panel Server
    listen 80;
    server_name admin.main_domain;

    root /var/www/admin_html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

#### Step 5: Test and Restart Nginx

1. Test the Nginx configuration:
   ```bash
   sudo nginx -t
   ```
2. Restart Nginx:
   ```bash
   sudo systemctl restart nginx
   ```
3. Enable Nginx on boot:
   ```bash
   sudo systemctl enable nginx
   ```

#### Step 6: Configure the Firewall

1. Allow essential ports:
   ```bash
   sudo ufw allow 80
   sudo ufw allow 443
   sudo ufw enable
   ```

#### Step 7: Install SSL Certificates Using Let's Encrypt

1. Install Certbot:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```
2. Obtain SSL certificates:
   ```bash
   sudo certbot --nginx -d main_domain -d www.main_domain -d admin.main_domain
   ```
3. Test SSL auto-renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

---

## Deployment Script

The `build` script in the root `package.json` automates the deployment process:
```json
"build": "cd frontend && npm install && npm run build && sudo cp -r dist/* /var/www/html && cd .. && cd Admin && npm install && npm run build && sudo cp -r dist/* /var/www/admin_html && cd .. && cd Backend && npm install && pm2 start server.js --watch"
```

To start the application for development:
```json
"start": "concurrently \"cd frontend && npm start\" \"cd Admin && npm start\" \"cd Backend && nodemon server.js\""
```

---

## Additional Notes

1. Ensure all domains (`main_domain`, `admin.main_domain`, etc.) point to your VPS IP.
2. Use PM2 to manage backend processes.
3. Regularly update the server and dependencies.

---

For detailed explanations or troubleshooting, refer to the respective sections above.
