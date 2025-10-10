# LuxeGenie System Setup Documentation

## Overview
This guide provides step-by-step instructions for setting up the LuxeGenie application on a Radxa board with automatic startup on boot.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Initial Setup](#initial-setup)
3. [System Configuration](#system-configuration)
4. [Application Setup](#application-setup)
5. [Auto-Start Configuration](#auto-start-configuration)
6. [Verification and Testing](#verification-and-testing)
7. [Troubleshooting](#troubleshooting)
8. [Important Precautions](#important-precautions)

---

## Prerequisites

### Hardware Requirements
- Radxa Zero 3 board
- SD card (minimum 16GB recommended, Class 10 or UHS-I)
- Display connected via HDMI/DisplayPort
- Stable power supply (5V/3A recommended)
- Keyboard and mouse for initial setup
- Ethernet cable or WiFi for internet connectivity

### Software Requirements
- **OS Image**: Ubuntu 24.04 for Radxa Zero 3
  - Download: [ubuntu-24.04-preinstalled-desktop-arm64-radxa-zero3.img.xz](https://github.com/Joshua-Riek/ubuntu-rockchip/releases/download/v2.4.0/ubuntu-24.04-preinstalled-desktop-arm64-radxa-zero3.img.xz)
- LuxegenieV1.AppImage (frontend application)
- LuxegenieBackend executable
- Backend virtual environment with dependencies

---

## Initial Setup

### Step 1: Flash the SD Card

1. Download the Ubuntu 24.04 image for Radxa Zero 3:
   ```
   https://github.com/Joshua-Riek/ubuntu-rockchip/releases/download/v2.4.0/ubuntu-24.04-preinstalled-desktop-arm64-radxa-zero3.img.xz
   ```

2. Extract the `.xz` file:
   - **Linux/Mac**: `xz -d ubuntu-24.04-preinstalled-desktop-arm64-radxa-zero3.img.xz`
   - **Windows**: Use 7-Zip or similar tool

3. Flash the extracted `.img` file to your SD card using one of these tools:
   - **Balena Etcher** (Recommended for beginners): https://www.balena.io/etcher/
   - **Rufus** (Windows): https://rufus.ie/
   - **dd command** (Linux/Mac):
     ```bash
     sudo dd if=ubuntu-24.04-preinstalled-desktop-arm64-radxa-zero3.img of=/dev/sdX bs=4M status=progress
     # Replace /dev/sdX with your SD card device (e.g., /dev/sdb)
     ```

4. Insert the SD card into your Radxa Zero 3 board
5. Power on the board and complete the initial boot

**⚠️ Precautions:** 
- Ensure the SD card is properly formatted and the image is completely written before removing the card
- Verify the `.img.xz` file checksum after download to ensure integrity
- Double-check the device path when using `dd` to avoid overwriting the wrong disk
- Use a high-quality SD card (Class 10 or UHS-I minimum) for better performance

---

### Step 2: Configure the Operating System

During the initial setup:
- Create user account with username: **radxa**
- Set a secure password
- Complete any additional setup prompts

**⚠️ Precaution:** Use a strong password for security. Write it down in a secure location.

---

## System Configuration

### Step 3: Update System Packages

```bash
sudo apt update
```

This updates the package repository index.

### Step 4: Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

The `-y` flag automatically confirms the upgrade.

**⚠️ Precaution:** This may take 10-30 minutes depending on your internet speed. Ensure stable power and network connection.

---

### Step 5: Install X Window System

```bash
sudo apt install --no-install-recommends xorg xinit
```

This installs the minimal X11 display server components needed for graphical applications.

**⚠️ Precaution:** The `--no-install-recommends` flag keeps the installation minimal. Do not modify this unless you need additional X11 features.

---

### Step 6: Install Additional Dependencies

Install required dependencies for the frontend AppImage and backend:

```bash
sudo apt install zlib1g zlib1g-dev libfuse2t64 ruby ruby-dev build-essential fuse
```

These packages are required for:
- `libfuse2t64` & `fuse`: Running AppImage files (FUSE filesystem support)
- `zlib1g` & `zlib1g-dev`: Compression library and development files
- `ruby` & `ruby-dev`: Ruby runtime and development files
- `build-essential`: Compilation tools (gcc, g++, make, etc.)

**⚠️ Precaution:** All these dependencies are essential for the AppImage to run. Missing any of them will cause the frontend to fail.

---

## Application Setup

### Step 7: Create Directory Structure

```bash
mkdir /home/radxa/frontend
mkdir /home/radxa/backend
```

**⚠️ Precaution:** Use these exact paths. The systemd service and scripts reference these absolute paths.

---

### Step 8: Copy Application Files

Copy your application files to the appropriate directories:
- Place `LuxegenieV1.AppImage` in `/home/radxa/frontend/`
- Place `LuxegenieBackend` and related files in `/home/radxa/backend/`
- Ensure the backend virtual environment is at `/home/radxa/backend/venv/`

**⚠️ Precaution:** Verify all files are copied completely before proceeding.

---

### Step 9: Set Execute Permissions

```bash
chmod +x /home/radxa/frontend/LuxegenieV1.AppImage
chmod +x /home/radxa/backend/LuxegenieBackend
```

This makes the application files executable.

**⚠️ Precaution:** Only set execute permissions on trusted files. Verify the source of your applications.

---

## Auto-Start Configuration

### Step 10: Configure X Session Startup Script

Create the `.xinitrc` file:

```bash
nano ~/.xinitrc
```

Add the following content:

```bash
#!/bin/bash

# Wait for system to stabilize
sleep 2

# Activate Python virtual environment
source /home/radxa/backend/venv/bin/activate

# Wait for environment activation
sleep 2

# Start backend with logging
python3 /home/radxa/backend/main.py > /home/radxa/backend/backend.log 2>&1 &

# Wait for backend to initialize
sleep 5

# Start frontend application
exec /home/radxa/frontend/LuxegenieV1.AppImage
```

Save and exit (Ctrl+X, then Y, then Enter).

**⚠️ Precautions:**
- The `sleep` commands ensure proper initialization timing
- Adjust sleep durations if your system is slower or faster
- Backend logs are written to `/home/radxa/backend/backend.log`
- The `exec` command replaces the shell process with the AppImage

---

### Step 11: Create Systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/app.service
```

Add the following content:

```ini
[Unit]
Description=Autostart X and AppImage
After=network.target systemd-user-sessions.service getty@tty1.service
Wants=graphical.target

[Service]
User=radxa
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/radxa/.Xauthority
TTYPath=/dev/tty1
StandardInput=tty
StandardOutput=journal
StandardError=journal
ExecStart=/usr/bin/startx
Restart=on-failure

[Install]
WantedBy=graphical.target
```

Save and exit.

**⚠️ Critical Precautions:**
- **DISPLAY=:0** - This is the first X display. If you have multiple displays or sessions, this number may need to be adjusted
- **User=radxa** - Must match your username exactly
- **TTYPath=/dev/tty1** - Uses the first virtual terminal. Change if needed
- **XAUTHORITY path** - Must match your user's home directory

---

### Step 12: Configure X Wrapper

```bash
sudo nano /etc/X11/Xwrapper.config
```

Add or modify to include:

```
allowed_users=anybody
```

This allows non-root users to start X server.

**⚠️ Precaution:** This reduces security slightly. In production, consider using `allowed_users=console` instead.

---

### Step 13: Add User to Required Groups

```bash
sudo usermod -aG video,input radxa
```

This grants the radxa user access to video devices and input devices.

**⚠️ Precaution:** These permissions are necessary for graphics and input handling. Changes take effect after logout/login.

---

### Step 14: Reload Systemd Configuration

```bash
sudo systemctl daemon-reload
```

This reloads systemd to recognize the new service file.

---

### Step 15: Enable Auto-Start Service

```bash
sudo systemctl enable app.service
```

This configures the service to start automatically on boot.

---

## Verification and Testing

### Step 16: Start the Service Manually

```bash
sudo systemctl start app.service
```

This starts the service immediately for testing.

---

### Step 17: Check Service Status

```bash
sudo systemctl status app.service
```

Expected output should show:
- **Active: active (running)** - Service is running successfully
- No error messages in recent logs

**⚠️ Precaution:** If you see errors, check the troubleshooting section before rebooting.

---

### Step 18: Reboot and Verify Auto-Start

```bash
sudo reboot
```

After reboot, the system should automatically:
1. Start X server
2. Launch the backend service
3. Display the LuxeGenie frontend

---

## Troubleshooting

### Service Fails to Start

Check detailed logs:
```bash
sudo journalctl -u app.service -n 50
```

### Backend Issues

View backend logs:
```bash
cat /home/radxa/backend/backend.log
```

### X Server Issues

Check X server logs:
```bash
cat /home/radxa/.local/share/xorg/Xorg.0.log
```

### Display Number Issues

If your system uses a different display number:
1. Check running X displays: `ls /tmp/.X11-unix/`
2. Update `DISPLAY=:0` in `app.service` to match (e.g., `DISPLAY=:1`)

### Permission Issues

Verify file permissions:
```bash
ls -la /home/radxa/frontend/LuxegenieV1.AppImage
ls -la /home/radxa/backend/LuxegenieBackend
```

Both should show `-rwxr-xr-x` or similar with execute permissions.

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Black screen on boot | Check X server logs, verify display connection |
| Backend not starting | Verify virtual environment path and Python dependencies |
| Permission denied errors | Re-run chmod commands and verify user groups |
| Service starts then stops | Check sleep timings in .xinitrc, increase if needed |
| AppImage won't run | Ensure libfuse2t64 is installed, check AppImage integrity |

---

## Important Precautions

### ⚠️ Critical Configuration Items

1. **Display Number (DISPLAY=:0)**
   - Most systems use `:0` for the first display
   - Multi-seat setups may use `:1`, `:2`, etc.
   - Verify with `echo $DISPLAY` in a working X session

2. **File Paths**
   - All paths must be absolute (starting with `/`)
   - Do not use `~` in systemd service files
   - Verify paths match exactly: `/home/radxa/` not `/home/Radxa/`

3. **Timing Issues**
   - `sleep` commands prevent race conditions
   - Increase sleep durations on slower systems
   - Backend must fully initialize before frontend starts

4. **User Permissions**
   - User must be in `video` and `input` groups
   - Changes require logout/login or reboot
   - Service must run as correct user

5. **Virtual Environment**
   - Backend virtual environment must exist at specified path
   - All Python dependencies must be installed in venv
   - Activate venv before running Python scripts

### 🔒 Security Considerations

- Use strong passwords for the radxa user
- Keep system packages updated regularly
- Consider firewall rules if network-exposed
- Review application permissions periodically
- Backup configuration files before major changes

### 💾 Backup Recommendations

Before major changes, backup:
```bash
# Backup service file
sudo cp /etc/systemd/system/app.service /etc/systemd/system/app.service.backup

# Backup xinitrc
cp ~/.xinitrc ~/.xinitrc.backup

# Backup X wrapper config
sudo cp /etc/X11/Xwrapper.config /etc/X11/Xwrapper.config.backup
```

---

## Quick Reference Commands

```bash
# Check service status
sudo systemctl status app.service

# View service logs
sudo journalctl -u app.service -f

# Restart service
sudo systemctl restart app.service

# Stop service
sudo systemctl stop app.service

# Disable auto-start
sudo systemctl disable app.service

# View backend logs
tail -f /home/radxa/backend/backend.log
```

---

## Support and Maintenance

### Regular Maintenance

- Review logs weekly for errors
- Update system packages monthly
- Monitor disk space usage
- Verify backup integrity

### Getting Help

If you encounter issues not covered in troubleshooting:
1. Collect relevant logs (systemd, backend, X server)
2. Note the exact step where the issue occurs
3. Document any error messages verbatim
4. Check system resource usage (CPU, RAM, disk)

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Initial | First release of documentation |

---

**Document Prepared For:** LuxeGenie System Setup  
**Target Platform:** Radxa Board with Debian-based OS  
**Last Updated:** October 2025
