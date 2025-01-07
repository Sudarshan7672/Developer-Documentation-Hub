# VPS Hosting Setup for Ubuntu OS

This guide outlines how to host your project on a VPS running Ubuntu, including setting up the frontend, admin panel, and backend. We will serve the frontend and admin panels from specific paths, `/var/www/html` and `/var/www/admin_html`, respectively.

---

## Key Steps Overview:

- **Step 1**: Connect to your VPS and install necessary dependencies.
- **Step 2**: Generate and add SSH keys to GitHub for private repository access.
- **Step 3**: Clone the repository from GitHub and set up your environment.
- **Step 4**: Configure Nginx to serve the frontend and admin panels.
- **Step 5**: Test Nginx configuration, restart Nginx, and enable it.
- **Step 6**: Set up the firewall and enable essential ports.

---

## Step 1: Connect to Your VPS

### SSH into Your VPS

```bash
ssh root@your-vps-ip
```

### Update Your System and Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git nodejs npm nginx
```
Check if Nginx is installed successfully:
```bash
sudo systemctl status nginx
```
## Step 2: Generate SSH Key and Add to GitHub
### Generate a New SSH Key Pair
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```
Save the files with the name `id_ed25519` and `id_ed25519.pub` 
```
cat id_ed25519  # Check the private key
cat id_ed25519.pub  # Check the public key
```
### Set Permissions:
Ensure that your private key has the correct permissions:
```
chmod 600 id_ed25519
```

### Add the SSH key to the SSH agent:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/id_ed25519 # Adjust path if necessary to the private key
```
### Add Your Public Key to GitHub:
Copy the contents of your public key and add it to your GitHub account:
```
cat id_ed25519.pub
```
Go to GitHub → `Settings`→ `SSH and GPG Keys `→ `New SSH Key`, then paste the public key and click `Add SSH Key`.

### Test Your Connection:
Finally, test your SSH connection again:
```
ssh -T git@github.com
```

## Step 3: Clone the Repository
### Clone Your Repository
Now, clone the Main_Folder repository, which contains all three sub-repositories:
```bash
git clone git@github.com:your-username/your-repository.git
cd your-repository
```

## Step 4: Configure Nginx to Serve Frontend and Admin Panels
Open the Nginx default configuration file to set up the server for the frontend, backend, and admin panels:
```bash
sudo nano /etc/nginx/sites-available/default
```
Replace the contents with the following:
```
##
# Virtual Host Configs
##

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
        proxy_pass http://localhost:8080;  # Backend running on localhost:8080
        set $allowed_origin "";
        if ($http_origin ~* (https://main_domain|https://admin.main_domain.com)) {
            set $allowed_origin $http_origin;
        }
        add_header Access-Control-Allow-Origin $allowed_origin always;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Content-Type, Authorization";
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

# Optional: Logging Configuration
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```
### Note - Replace the `main_domain` with the `actual domain` name.

## Build and Deploy the Application
Use `npm start` from the `Main_Folder` directory to start the application and deploy it automatically to the correct directories:
```bash
npm start
```
Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html
sudo chown -R www-data:www-data /var/www/admin_html
sudo chmod -R 755 /var/www/html
sudo chmod -R 755 /var/www/admin_html
```

### pm2 Configurations
Install
```bash
sudo npm install pm2 -g
```
Start the app using pm2:
```bash
pm2 start npm --name "main-app" -- start
```
To ensure it starts on reboot:
```bash
pm2 startup
pm2 save
```
```bash
pm2 restart main-app
```

## Step 5: Test and Restart Nginx
##### If the configuration is successful, you will see the message nginx: configuration file /etc/nginx/nginx.conf test is successful.

```bash
sudo nginx -t
```


Error Logs Of NginX
```bash
sudo tail -f /var/log/nginx/error.log
```


### Enable the Site Configuration
Create symbolic links to enable the sites:
```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

Restart Nginx to apply the changes:
```bash
sudo systemctl restart nginx
```

Enable Nginx to Start on Boot
```bash
sudo systemctl enable nginx
```
## Step 6: Configure the Firewall (Optional)
```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## Step 7: Install SSL Certificates Using Let's Encrypt
### Install Certbot (Let's Encrypt Client)
```bash
sudo apt install certbot python3-certbot-nginx
```
### Obtain SSL Certificate
```bash
sudo certbot --nginx -d main_domain -d www.main_domain -d admin.main_domain
```
```bash
sudo nginx -t
```
```bash
sudo systemctl restart nginx
```
### Set Up Auto-Renewal for SSL Certificates
```bash
sudo certbot renew --dry-run
sudo crontab -e
0 0 * * * certbot renew --quiet && systemctl reload nginx
```









