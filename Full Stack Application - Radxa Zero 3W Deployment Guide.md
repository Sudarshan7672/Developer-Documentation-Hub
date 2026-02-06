# IOT Application Deployment Guide for Radxa Zero 3W
## Required - Electron Build(ARM64), Python Portable Backend(Pyinstaller)
Complete documentation for deploying the IOT frontend and backend on Radxa Zero 3W running official Debian OS.

**Working Directory:** `/home/radxa`

---

## Table of Contents
1. [System Prerequisites](#1-system-prerequisites)
2. [Frontend Kiosk Setup](#2-frontend-kiosk-setup)
3. [Backend Service Setup](#3-backend-service-setup)
4. [Auto-Update Service](#4-auto-update-service)
5. [System Configuration](#5-system-configuration)
6. [Final Deployment Steps](#6-final-deployment-steps)
7. [Debugging & Troubleshooting](#7-debugging--troubleshooting)

---

## 1. System Prerequisites

### Install Required Packages
Install all necessary system packages for running the kiosk and AppImage:

```bash
sudo apt update
sudo apt install -y \
  unclutter \
  openbox \
  dbus-x11 \
  xorg \
  xinit \
  xterm \
  libfuse2 \
  ruby \
  ruby-dev \
  zlib1g \
  zlib1g-dev \
  build-essential \
  mesa-utils \
  fuse \
  --no-install-recommends
```

**What these packages do:**
- `unclutter` - Hides the mouse cursor in kiosk mode
- `openbox` - Lightweight window manager
- `xorg/xinit` - X Window System for displaying the GUI
- `libfuse2/fuse` - Required to run AppImage files
- `dbus-x11` - Inter-process communication for Electron apps

### Add User to Required Groups
Grant the `radxa` user access to display, serial ports, and input devices:

```bash
sudo usermod -aG tty,video,dialout,input,render radxa
```

---

## 2. Frontend Kiosk Setup

### Step 1: Create Startup Script
This script launches your frontend AppImage in kiosk mode with proper display settings.

```bash
nano /home/radxa/start-app.sh
```

**Add this content:**

```bash
#!/bin/bash
# Frontend Kiosk Launcher for LuxeGenie
# Logs all activity to /home/radxa/radxa_startup.log

LOGFILE="/home/radxa/radxa_startup.log"

echo "-------------------------------------------------" >> "$LOGFILE"
echo "$(date -Is) [INFO] Application startup initiated" >> "$LOGFILE"

# Set display environment
export DISPLAY=:0

# DBus session for Electron (required for AppImage)
export XDG_RUNTIME_DIR=/run/user/1000
export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus

# Wait for X server to be ready (max 10 seconds)
for i in {1..10}; do
    if [ -S /tmp/.X11-unix/X0 ]; then
        echo "$(date -Is) [INFO] X server ready" >> "$LOGFILE"
        break
    fi
    echo "$(date -Is) [INFO] Waiting for X server..." >> "$LOGFILE"
    sleep 1
done

# Disable screen blanking and power saving
xset s off >> "$LOGFILE" 2>&1
xset -dpms >> "$LOGFILE" 2>&1
xset s noblank >> "$LOGFILE" 2>&1

# Hide mouse cursor after 0.1 seconds of inactivity
unclutter -idle 0.1 -root >> "$LOGFILE" 2>&1 &

# Set black background
xsetroot -solid black >> "$LOGFILE" 2>&1

# Brief delay for system stability
sleep 2

echo "$(date -Is) [INFO] Launching frontend AppImage" >> "$LOGFILE"

# Launch Electron AppImage with hardware acceleration disabled
exec /home/radxa/frontend/current_frontend.AppImage \
  --use-gl=egl \
  --enable-gpu-rasterization \
  --no-sandbox
  >> "$LOGFILE" 2>&1
```

**Make it executable:**

```bash
chmod +x /home/radxa/start-app.sh
```

### Step 2: Create X Session Configuration
Configure what happens when X server starts.

```bash
nano /home/radxa/.xinitrc
```

**Add this content:**

```bash
#!/bin/sh
# X initialization script

# Disable all power saving features
xset -dpms
xset s off
xset s noblank

# Set black background
xsetroot -solid black

# Launch the kiosk application
exec /home/radxa/start-app.sh
```

**Make it executable:**

```bash
chmod +x /home/radxa/.xinitrc
```

### Step 3: Create Kiosk Systemd Service
This service automatically starts your kiosk on boot.

```bash
sudo nano /etc/systemd/system/kiosk.service
```

**Add this content:**

```bash
[Unit]
Description=LuxeGenie Kiosk Mode (Frontend)
After=systemd-user-sessions.service
Conflicts=getty@tty1.service

[Service]
User=radxa
WorkingDirectory=/home/radxa
Environment=XAUTHORITY=/home/radxa/.Xauthority
Environment=DISPLAY=:0
Environment=LIBGL_ALWAYS_SOFTWARE=1
StandardInput=tty-force
TTYPath=/dev/tty1
StandardOutput=journal

# Start X server with the .xinitrc configuration
ExecStart=/usr/bin/xinit /home/radxa/.xinitrc -- :0 vt1 -keeptty -nolisten tcp -nocursor

# Restart on failure
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Step 4: Create Log File
Initialize the log file with correct permissions:

```bash
touch /home/radxa/radxa_startup.log
chmod 644 /home/radxa/radxa_startup.log
```

---

## 3. Backend Service Setup

The backend runs as root because it needs to control WiFi using `nmcli`.

### Create Backend Service

```bash
sudo nano /etc/systemd/system/backend.service
```

**Add this content:**

```bash
[Unit]
Description=LuxeGenie Backend Service
After=NetworkManager.service
Wants=NetworkManager.service

[Service]
# Run as root to control WiFi/network settings
User=root

# Backend executable location
WorkingDirectory=/home/radxa

# Path to your backend binary
ExecStart=/home/radxa/backend/current_backend

# Ensure system commands are in PATH
Environment="PATH=/usr/sbin:/usr/bin:/sbin:/bin"

# Auto-restart on crash (with 2-second delay)
Restart=on-failure
RestartSec=2

[Install]
WantedBy=multi-user.target
```

---

## 4. Auto-Update Service(OTA)

This service checks for updates 5 minutes after boot and applies them automatically.

### Create Update Service

```bash
sudo nano /etc/systemd/system/update.service
```

**Add this content:**

```bash
[Unit]
Description=Over-The-Air-Update (Delayed after boot)
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=root
Group=root

# Wait 5 minutes (300 seconds) after boot before running
ExecStartPre=/bin/sleep 300

# Run the update script
ExecStart=/bin/bash /home/radxa/scripts/update.sh

[Install]
WantedBy=multi-user.target
```

**Note:** You need to create the actual update script at `/home/radxa/scripts/update.sh`

**Create The Update Script**
```bash
mkdir scripts
cd scripts
sudo nano update.sh
```
**Add this content:**
```bash
#!/bin/bash

# LuxeGenie Update Script
# This script checks for updates and manages the update process

set -e

# Configuration
UPDATE_URL="https://api.mywoobly.com/api/v1/luxegenie/updates/check-update/latest"
VERSION_FILE="/home/radxa/.version"
LOG_FILE="/var/log/luxegenie-update.log"
FRONTEND_DIR="/home/radxa/frontend"
BACKEND_DIR="/home/radxa/backend"
BACKUP_DIR="/home/radxa/backup"
FRONTEND_BACKUP_DIR="$BACKUP_DIR/frontend"
BACKEND_BACKUP_DIR="$BACKUP_DIR/backend"
TEMP_DIR="/tmp/luxegenie_update"
SERVICE_NAME="app.service"
FRONTEND_SERVICE="kiosk.service"
BACKEND_SERVICE="backend.service"
SPLASH_IMAGE="/home/radxa/splash/update.png"
FB_SPLASH_PID=""
ACTIVE_TTY=""
SPLASH_TTY=""


# Logging function
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Error handling function
error_exit() {
    log "ERROR: $1"
    cleanup
    exit 1
}

# Cleanup function
cleanup() {
    log "Cleaning up temporary files..."
    rm -rf "$TEMP_DIR"
}

# Rollback function
rollback() {
    log "Rolling back to previous version..."
    
    # Restore frontend
    if [ -f "$FRONTEND_BACKUP_DIR/previous_frontend.AppImage" ]; then
        log "Restoring frontend from backup..."
        cp "$FRONTEND_BACKUP_DIR/previous_frontend.AppImage" "$FRONTEND_DIR/current_frontend.AppImage"
        chmod +x "$FRONTEND_DIR/current_frontend.AppImage"
        log "Frontend restored successfully"
    else
        log "WARNING: No frontend backup found to restore"
    fi
    
    # Restore backend
    if [ -f "$BACKEND_BACKUP_DIR/previous_backend" ]; then
        log "Restoring backend from backup..."
        cp "$BACKEND_BACKUP_DIR/previous_backend" "$BACKEND_DIR/current_backend"
        chmod +x "$BACKEND_DIR/current_backend"
        log "Backend restored successfully"
    else
        log "WARNING: No backend backup found to restore"
    fi
    
    log "Rollback completed"
}

# Check internet connectivity
check_internet() {
    log "Checking internet connectivity..."
    if ! ping -c 1 8.8.8.8 &> /dev/null; then
        log "No internet connection detected. Exiting."
        exit 0
    fi
    log "Internet connection verified"
}

# Create necessary directories
create_directories() {
    log "Ensuring required directories exist..."
    mkdir -p "$FRONTEND_DIR"
    mkdir -p "$BACKEND_DIR"
    mkdir -p "$FRONTEND_BACKUP_DIR"
    mkdir -p "$BACKEND_BACKUP_DIR"
    mkdir -p "$TEMP_DIR"
    mkdir -p "$(dirname "$SPLASH_HTML")"
    log "Directories verified/created"
}

# Get local version
get_local_version() {
    if [ -f "$VERSION_FILE" ]; then
        LOCAL_VERSION=$(cat "$VERSION_FILE")
        log "Local version: $LOCAL_VERSION"
    else
        log "Version file not found, creating with version 0.0.0"
        echo "0.0.0" > "$VERSION_FILE"
        LOCAL_VERSION="0.0.0"
    fi
}

# Check for updates
check_updates() {
    log "Checking for updates from server..."
    
    RESPONSE=$(curl -s "$UPDATE_URL")
    
    if [ -z "$RESPONSE" ]; then
        error_exit "Failed to fetch update information from server"
    fi
    
    log "Update response received"
    
    REMOTE_VERSION=$(echo "$RESPONSE" | grep -o '"version":"[^"]*"' | head -1 | cut -d'"' -f4)
    FRONTEND_URL=$(echo "$RESPONSE" | grep -o '"frontend_url":"[^"]*"' | cut -d'"' -f4)
    BACKEND_URL=$(echo "$RESPONSE" | grep -o '"backend_url":"[^"]*"' | cut -d'"' -f4)
    FRONTEND_CHECKSUM=$(echo "$RESPONSE" | grep -o '"frontend_checksum":"[^"]*"' | cut -d'"' -f4)
    BACKEND_CHECKSUM=$(echo "$RESPONSE" | grep -o '"backend_checksum":"[^"]*"' | cut -d'"' -f4)
    
    log "Remote version: $REMOTE_VERSION"
    
    if [ "$LOCAL_VERSION" = "$REMOTE_VERSION" ]; then
        log "Already running the latest version ($LOCAL_VERSION). No update needed."
        cleanup
        exit 0
    fi
    
    log "New version available. Starting update process..."
}

# Backup current files
backup_current() {
    log "Backing up current files..."
    
    if [ -f "$FRONTEND_DIR/current_frontend.AppImage" ]; then
        log "Backing up frontend..."
        cp "$FRONTEND_DIR/current_frontend.AppImage" "$FRONTEND_BACKUP_DIR/previous_frontend.AppImage"
        log "Frontend backed up successfully"
    else
        log "No existing frontend to backup"
    fi
    
    if [ -f "$BACKEND_DIR/current_backend" ]; then
        log "Backing up backend..."
        cp "$BACKEND_DIR/current_backend" "$BACKEND_BACKUP_DIR/previous_backend"
        log "Backend backed up successfully"
    else
        log "No existing backend to backup"
    fi
}

# Download and verify frontend
download_frontend() {
    log "Downloading frontend from: $FRONTEND_URL"
    
    cd "$TEMP_DIR"
    
    if ! wget -q --show-progress "$FRONTEND_URL" -O frontend.tar.gz; then
        error_exit "Failed to download frontend"
    fi
    
    log "Frontend downloaded successfully"
    
    log "Extracting frontend..."
    if ! tar -xzf frontend.tar.gz; then
        error_exit "Failed to extract frontend archive"
    fi
    
    FRONTEND_FILE=$(find . -maxdepth 1 -name "*.AppImage" -type f | head -1)
    
    if [ -z "$FRONTEND_FILE" ]; then
        error_exit "No AppImage file found in extracted frontend archive"
    fi
    
    log "Frontend extracted: $FRONTEND_FILE"
    
    log "Verifying frontend checksum..."
    CALCULATED_CHECKSUM=$(sha256sum "$FRONTEND_FILE" | awk '{print $1}')
    
    if [ "$CALCULATED_CHECKSUM" != "$FRONTEND_CHECKSUM" ]; then
        log "Checksum mismatch!"
        log "Expected: $FRONTEND_CHECKSUM"
        log "Got: $CALCULATED_CHECKSUM"
        error_exit "Frontend checksum verification failed"
    fi
    
    log "Frontend checksum verified successfully"
    
    log "Installing new frontend..."
    cp "$FRONTEND_FILE" "$FRONTEND_DIR/current_frontend.AppImage"
    chmod +x "$FRONTEND_DIR/current_frontend.AppImage"
    log "Frontend installed successfully"
}

# Download and verify backend
download_backend() {
    log "Downloading backend from: $BACKEND_URL"

    BACKEND_TMP_DIR="$TEMP_DIR/backend"
    rm -rf "$BACKEND_TMP_DIR"
    mkdir -p "$BACKEND_TMP_DIR"
    cd "$BACKEND_TMP_DIR"

    if ! wget -q --show-progress "$BACKEND_URL" -O backend.tar.gz; then
        error_exit "Failed to download backend"
    fi
    log "Backend downloaded successfully"

    log "Extracting backend archive..."
    if ! tar -xzf backend.tar.gz; then
        error_exit "Failed to extract backend archive"
    fi

    BACKEND_FILE=$(find . -maxdepth 1 -type f -exec file {} \; \
        | grep -E 'ELF .* executable' \
        | grep -v 'AppImage' \
        | cut -d: -f1 \
        | head -1)

    if [ -z "$BACKEND_FILE" ]; then
        error_exit "No backend executable found after extraction"
    fi

    log "Backend executable found: $BACKEND_FILE"

    log "Verifying backend checksum..."
    CALCULATED_CHECKSUM=$(sha256sum "$BACKEND_FILE" | awk '{print $1}')

    if [ "$CALCULATED_CHECKSUM" != "$BACKEND_CHECKSUM" ]; then
        log "Checksum mismatch!"
        log "Expected: $BACKEND_CHECKSUM"
        log "Got: $CALCULATED_CHECKSUM"
        error_exit "Backend checksum verification failed"
    fi

    log "Backend checksum verified successfully"

    log "Installing new backend..."
    cp "$BACKEND_FILE" "$BACKEND_DIR/current_backend"
    chmod +x "$BACKEND_DIR/current_backend"
    log "Backend installed successfully"
}

# Update version file
update_version_file() {
    log "Updating version file to $REMOTE_VERSION"
    echo "$REMOTE_VERSION" > "$VERSION_FILE"
    log "Version file updated"
}

# Stop services before update
stop_services() {
    log "Stopping services before update..."

    if systemctl list-unit-files | grep -q "$FRONTEND_SERVICE"; then
        log "Stopping $FRONTEND_SERVICE..."
        systemctl stop "$FRONTEND_SERVICE" && log "$FRONTEND_SERVICE stopped successfully" \
        || log "WARNING: Failed to stop $FRONTEND_SERVICE"
    fi

    if systemctl list-unit-files | grep -q "$BACKEND_SERVICE"; then
        log "Stopping $BACKEND_SERVICE..."
        systemctl stop "$BACKEND_SERVICE" && log "$BACKEND_SERVICE stopped successfully" \
        || log "WARNING: Failed to stop $BACKEND_SERVICE"
    fi

    sleep 2
}

# Restart services
restart_services() {
    log "Restarting services..."

    if systemctl list-unit-files | grep -q "$BACKEND_SERVICE"; then
        log "Restarting $BACKEND_SERVICE..."
        systemctl restart "$BACKEND_SERVICE" && log "$BACKEND_SERVICE restarted successfully" \
        || log "WARNING: Failed to restart $BACKEND_SERVICE"
    else
        log "WARNING: $BACKEND_SERVICE not found"
    fi

    if systemctl list-unit-files | grep -q "$FRONTEND_SERVICE"; then
        log "Restarting $FRONTEND_SERVICE..."
        systemctl restart "$FRONTEND_SERVICE" && log "$FRONTEND_SERVICE restarted successfully" \
        || log "WARNING: Failed to restart $FRONTEND_SERVICE"
    else
        log "WARNING: $FRONTEND_SERVICE not found"
    fi
}

# Show splash screen
show_splash() {
    log "Preparing framebuffer splash..."

    if [ ! -f "$SPLASH_IMAGE" ]; then
        log "WARNING: Splash image not found"
        return 0
    fi

    detect_active_tty

    SPLASH_TTY=2
    [ "$ACTIVE_TTY" = "2" ] && SPLASH_TTY=3

    log "Switching to splash TTY: tty$SPLASH_TTY"
    chvt "$SPLASH_TTY" 2>/dev/null || true

    pkill -f fbi 2>/dev/null || true

    # Keep fbi alive explicitly
    (
        while true; do
            fbi -T "$SPLASH_TTY" -d /dev/fb0 \
                --noverbose \
                --autozoom \
                "$SPLASH_IMAGE" \
                </dev/tty"$SPLASH_TTY" >/dev/null 2>&1
            sleep 1
        done
    ) &

    FB_SPLASH_PID=$!
    log "Splash locked on tty$SPLASH_TTY (PID: $FB_SPLASH_PID)"
}


# Hide splash screen
hide_splash() {
    log "Hiding splash screen..."

    [ -n "$FB_SPLASH_PID" ] && kill "$FB_SPLASH_PID" 2>/dev/null || true
    pkill -f fbi 2>/dev/null || true

    if [ -n "$ACTIVE_TTY" ]; then
        log "Restoring original TTY: tty$ACTIVE_TTY"
        chvt "$ACTIVE_TTY" 2>/dev/null || true
    fi
}

# Verify Services
verify_services() {
    log "Verifying services health..."

    systemctl is-active --quiet "$BACKEND_SERVICE" \
        || error_exit "Backend service is not running"

    systemctl is-active --quiet "$FRONTEND_SERVICE" \
        || error_exit "Frontend service is not running"

    log "All services are running correctly"
}

detect_active_tty() {
    ACTIVE_TTY=$(fgconsole 2>/dev/null || echo 1)
    log "Active display TTY: tty$ACTIVE_TTY"
}

reboot_device() {
    log "Rebooting device to apply update..."
    sync
    sleep 2
    reboot
}



# Main execution
main() {
    log "=========================================="
    log "LuxeGenie Update Script Started"
    log "=========================================="
    
    check_internet
    create_directories
    get_local_version
    check_updates
    
    # Show splash screen BEFORE stopping services
    show_splash
    
    backup_current
    stop_services
    
    trap 'log "Update failed. Rolling back..."; hide_splash; rollback; reboot_device' ERR
    
    log "Downloading frontend..."
    download_frontend
    
    log "Downloading backend..."
    download_backend
    
    log "Finalizing update..."
    trap - ERR
    
    update_version_file
    
    log "Restarting services..."
    hide_splash
    
    restart_services
    verify_services
    cleanup
    reboot_device
    
    log "=========================================="
    log "Update completed successfully!"
    log "Updated from version $LOCAL_VERSION to $REMOTE_VERSION"
    log "=========================================="
}

# Run main function
main
```
**Update the Permission for update scritp**
```bash
sudo chmod +x update.sh
cd ..
```
**Create a directory and add the update splash image**
```bash
sudo mkdir -p /home/radxa/splash
sudo chown -R radxa:radxa /home/radxa/splash
```
Example Path
```bash
/home/radxa/splash/update.png
```
Copy the image from your local machine to the remote via ssh
```bash
scp /path/to/update.png radxa@<RADXA_IP>:/home/radxa/splash/update.png
```
Example
```bash
scp ~/Downloads/update.png radxa@192.168.1.:/home/radxa/splash/update.png
```

**Sometime the Ping is not permitted on Some OS. For ping permissions run this comnmand**
```bash
sudo setcap cap_net_rawtp /bin/ping
```
---

## 5. System Configuration

### Configure X Server Permissions

Allow any user to start X server:

```bash
sudo nano /etc/X11/Xwrapper.config
```

**Add:**

```
allowed_users=anybody
needs_root_rights=yes
```

### Enable Auto-Login on TTY1

This logs in the `radxa` user automatically on console boot.

```bash
sudo systemctl edit getty@tty1
```

**Add this content:**

```bash
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin radxa --noclear %I $TERM
```

### Auto-Start X on Login

Configure the user profile to start X automatically when logging in on TTY1.

```bash
nano /home/radxa/.bash_profile
```

**Add at the end:**

```bash
# Auto-start X if on tty1 and not already running
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
    exec startx -- -nocursor
fi
```


### Enable UART Communication

If your hardware uses UART for communication:
If Using the Radxa's Official Image
```bash
sudo resetup
```

Then navigate: **Overlays → Manage Overlays → Select UART2 M0 → Exit**

---

If using third party armbian Image
```bash
sudo nano /boot/armbianEnv.txt
```
add
```bash
overlays=uart1
```
---

If using the rockchip-ubuntu image then manage the overlay tree manually
Follow the steps - 
**Identify the active DTB file**
```bash
uname -r
ls /lib/firmware/$(uname -r)/device-tree/rockchip | grep radxa-zero3
```
You should see something like this
```bash
rk3566-radxa-zero3.dtb
```
**Back up the original DTB (IMPORTANT)**
```bash
sudo cp /lib/firmware/$(uname -r)/device-tree/rockchip/rk3566-radxa-zero3.dtb \
/lib/firmware/$(uname -r)/device-tree/rockchip/rk3566-radxa-zero3.dtb.bak
```
**Convert DTB → DTS**
```bash
dtc -I dtb -O dts \
-o ~/rk3566-radxa-zero3.dts \
/lib/firmware/$(uname -r)/device-tree/rockchip/rk3566-radxa-zero3.dtb
```
**Edit the DTS to enable UART2 (pins 6/8/10)**
```bash
nano ~/rk3566-radxa-zero3.dts
```
**Find this node:**
```bash
serial@fe660000 {
```
**edit it to exactly this:**
```bash
serial@fe660000 {
    compatible = "rockchip,rk3568-uart", "snps,dw-apb-uart";
    reg = <0x00 0xfe660000 0x00 0x100>;
    interrupts = <0x00 0x76 0x04>;
    clocks = <0x23 0x120 0x23 0x11c>;
    clock-names = "baudclk", "apb_pclk";
    reg-shift = <0x02>;
    reg-io-width = <0x04>;
    pinctrl-names = "default";
    pinctrl-0 = <0xe9>;   /* uart2m0-xfer */
    status = "okay";
};
```
**Compile DTS → DTB**
```bash
dtc -I dts -O dtb \
-o ~/rk3566-radxa-zero3-fixed.dtb \
~/rk3566-radxa-zero3.dts
```
**Install the fixed DTB**
```bash
sudo cp ~/rk3566-radxa-zero3-fixed.dtb \
/lib/firmware/$(uname -r)/device-tree/rockchip/rk3566-radxa-zero3.dtb
```
**Remove UART console from boot args (CRITICAL)**
```bash
sudo nano /boot/extlinux/extlinux.conf
```
```bash
Change:
console=ttyS2,1500000 console=tty1
To:
console=tty1
```

## 6. Final Deployment Steps

After completing all configuration:

1. **Reload systemd:**
   ```bash
   sudo systemctl daemon-reload
   ```

2. **Enable all services:**
   ```bash
   sudo systemctl enable kiosk.service
   sudo systemctl enable backend.service
   sudo systemctl enable update.service
   ```

3. **Disable Getty Service**
   ```bash
   sudo systemctl disable getty@tty1.service
   ```

4. **Disable the Getty Service to avoid ttyS' Port Conflict**
```bash
sudo systemctl stop serial-getty@ttyS2.service
sudo systemctl disable serial-getty@ttyS2.service
sudo systemctl mask serial-getty@ttyS2.service
```
5. **Disable Display Manager (if installed)**

If you have LightDM or another display manager running, disable it:

```bash
sudo systemctl stop lightdm
sudo systemctl disable lightdm
```
If you have gdm3 or another display manager running, disable it:

```bash
sudo systemctl stop gdm3
sudo systemctl disable gdm3
```
   
6. **Complete System Reboot**

```bash
sudo reboot
```

The system will now automatically:
- Boot into kiosk mode with your frontend AppImage
- Start the backend service with WiFi control
- Check for updates 5 minutes after each boot

---

## 7. Debugging & Troubleshooting

### View Kiosk Service Logs (Real-time)

```bash
journalctl -u kiosk.service -f
```

### View Backend Service Logs (Real-time)

```bash
journalctl -u backend.service -f
```

### Check Frontend Startup Log

```bash
tail -f /home/radxa/luxegenie_startup.log
```

### Check if X Server is Running

```bash
ps -ef | grep Xorg
```

### Restart Kiosk Service

```bash
sudo systemctl restart kiosk.service
```

### Restart Backend Service

```bash
sudo systemctl restart backend.service
```

### Restart Update Service

```bash
sudo systemctl restart update.service
```

### Check Service Status

```bash
sudo systemctl status kiosk.service
sudo systemctl status backend.service
sudo systemctl status update.service
```

### Check GPU is Initialized or Not
```bash
export DISPLAY=:0
glxinfo | grep -E "OpenGL renderer|OpenGL version"
```

### Watch GPU Load
```bash
watch -n 1 cat /sys/class/devfreq/*.gpu/load
```

### Check active virtual terminal (DISPLAY TTY)
```bash
fgconsole
```
### See all TTYs and what’s running on them
```bash
w
```
### To switch the screen back to tty1
```bash
sudo chvt 1
```

### If you get a permission error, run it explicitly as root:
```bash
sudo su
chvt 1
exit
```
---



## Notes

- Frontend AppImage location: `/home/radxa/frontend/current_frontend.AppImage`
- Backend binary location: `/home/radxa/backend/current_backend`
- Update script location: `/home/radxa/scripts/update.sh`
- All logs are stored in: `/home/radxa/radxa_startup.log` and system journal

Make sure these files exist and have execute permissions before enabling the services.
