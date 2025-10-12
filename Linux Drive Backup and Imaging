# Drive and OS Backup Guide using DD Command

A comprehensive guide for creating disk images and backups using the `dd` command in Linux, with Windows troubleshooting steps.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Linux Backup Process](#linux-backup-process)
- [Understanding DD Command Parameters](#understanding-dd-command-parameters)
- [Windows Troubleshooting](#windows-troubleshooting)
- [Safety Tips](#safety-tips)
- [Restoring from Backup](#restoring-from-backup)

---

## Overview

This guide explains how to create bit-perfect backups of drives, SD cards, or entire operating systems using the `dd` (data duplicator) command in Linux. These backups can be used for:
- Creating OS images for deployment
- Backing up SD cards (e.g., Raspberry Pi)
- Cloning entire drives
- Disaster recovery

---

## Prerequisites

### Required
- Linux system (Ubuntu, Debian, Fedora, etc.)
- Root/sudo access
- Sufficient storage space for the backup image
- Target drive or SD card to backup

### Optional
- Windows system (for accessing backup files)

---

## Linux Backup Process

### Step 1: Identify the Source Drive

First, list all available block devices to identify your source drive:

```bash
lsblk
```

**Output Example:**
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 465.8G  0 disk 
├─sda1        8:1    0   512M  0 part /boot/efi
└─sda2        8:2    0 465.3G  0 part /
sdb           8:16   1  29.7G  0 disk 
└─sdb1        8:17   1  29.7G  0 part /media/ubuntu/SDCARD
```

**Important:** Identify your device name (e.g., `/dev/sdb` for SD card, `/dev/sda` for hard drive)

⚠️ **Warning:** Double-check the device name! Using the wrong device will overwrite data.

---

### Step 2: Create the Backup Image

Use one of the following commands based on your needs:

#### For IMG Format (Disk Image)
```bash
sudo dd if=/dev/sdX of="/media/ubuntu/drivename/filename.img" bs=4M status=progress
```

#### For ISO Format (ISO Image)
```bash
sudo dd if=/dev/sdX of="/media/ubuntu/drivename/filename.iso" bs=4M status=progress
```

**Replace:**
- `sdX` with your actual device name (e.g., `sdb`)
- `drivename` with your backup drive mount point
- `filename` with your desired backup name

**Example:**
```bash
sudo dd if=/dev/sdb of="/media/ubuntu/backup_drive/raspberrypi_backup.img" bs=4M status=progress
```

---

## Understanding DD Command Parameters

### Command Breakdown

```bash
sudo dd if=/dev/sdX of="/path/to/backup.img" bs=4M status=progress
```

| Parameter | Meaning | Description |
|-----------|---------|-------------|
| `sudo` | Superuser Do | Executes command with root privileges (required for disk operations) |
| `dd` | Data Duplicator | The disk cloning utility |
| `if=` | Input File | Source device/file to read from (e.g., `/dev/sdb`) |
| `of=` | Output File | Destination file to write to (e.g., `/path/to/backup.img`) |
| `bs=` | Block Size | Amount of data read/written at once (e.g., `4M` = 4 megabytes) |
| `status=progress` | Status Display | Shows real-time progress of the operation |

### Block Size (bs) Explained

The `bs` parameter significantly affects backup speed:

- **`bs=4M`** (Recommended): Good balance of speed and compatibility
- **`bs=1M`**: Slower but more compatible with older systems
- **`bs=8M`** or **`bs=16M`**: Faster on modern systems with good hardware

**Tip:** Larger block sizes = faster transfers, but may cause issues on some systems.

### IMG vs ISO Format

- **`.img`**: Raw disk image containing exact copy of all data, partitions, and boot sectors
  - Use for: SD cards, USB drives, bootable media, OS backups
  
- **`.iso`**: Typically used for optical disc images (CDs/DVDs)
  - Use for: Creating bootable installation media
  - Note: For most drive backups, `.img` is more appropriate

---

## Windows Troubleshooting

### Issue: Backup File Not Visible or Accessible

If your backup image file doesn't appear in Windows File Explorer or can't be accessed:

#### Solution: Run CHKDSK (Check Disk)

1. **Open Command Prompt as Administrator**
   - Press `Win + X`
   - Select "Command Prompt (Admin)" or "Windows PowerShell (Admin)"

2. **Run CHKDSK Command:**
   ```cmd
   chkdsk D: /f
   ```

### CHKDSK Parameter Breakdown

```cmd
chkdsk D: /f
```

| Parameter | Meaning | Description |
|-----------|---------|-------------|
| `chkdsk` | Check Disk | Windows disk checking utility |
| `D:` | Drive Letter | The drive to check (replace with your actual drive letter) |
| `/f` | Fix Errors | Automatically fixes file system errors found |

### Additional CHKDSK Options

- **`/r`**: Locates bad sectors and recovers readable information (includes `/f`)
  ```cmd
  chkdsk D: /r
  ```
  
- **`/x`**: Forces the volume to dismount first (if needed)
  ```cmd
  chkdsk D: /f /x
  ```

**Note:** You may need to dismount the drive or restart Windows for CHKDSK to complete.

---

## Safety Tips

### Before Starting Backup

1. ✅ **Verify source device** using `lsblk` multiple times
2. ✅ **Ensure sufficient space** on destination drive
3. ✅ **Unmount the source** if it's mounted (except root partition):
   ```bash
   sudo umount /dev/sdX1
   ```
4. ✅ **Close all programs** using the source device
5. ✅ **Use absolute paths** for destination to avoid confusion

### During Backup

- ⏱️ **Be patient**: Large drives take hours to backup
- 🔌 **Don't disconnect** devices during the process
- 💾 **Monitor disk space** on destination drive
- 📊 **Watch progress** output for errors

### After Backup

- ✅ **Verify file size** matches expected size
- ✅ **Test restore** on a separate device if critical
- ✅ **Store securely** with proper labeling and date

---

## Restoring from Backup

To restore a backup image back to a drive:

```bash
sudo dd if="/media/ubuntu/drivename/filename.img" of=/dev/sdX bs=4M status=progress
```

⚠️ **Critical Warning:** This will **completely overwrite** the target drive!

**Verification after restore:**
```bash
sudo sync
lsblk
```

---

## Common Issues and Solutions

### Issue: "Permission Denied"
**Solution:** Ensure you're using `sudo` before the `dd` command

### Issue: "No space left on device"
**Solution:** Check destination drive has enough free space (at least equal to source size)

### Issue: Process seems frozen
**Solution:** DD doesn't show progress by default without `status=progress`. Wait patiently or check with:
```bash
sudo pkill -USR1 dd
```

### Issue: Backup file larger than expected
**Solution:** This is normal. `dd` creates exact disk images including empty space. To compress:
```bash
gzip filename.img
# Creates filename.img.gz (much smaller)
```

---

## Additional Resources

- [DD Man Page](https://man7.org/linux/man-pages/man1/dd.1.html)
- [Linux Block Devices Documentation](https://www.kernel.org/doc/html/latest/admin-guide/devices.html)
- [Ubuntu Community DD Guide](https://help.ubuntu.com/community/DriveImaging)

---

## License

This guide is released under the MIT License. Feel free to use, modify, and distribute.

## Contributing

Found an error or want to improve this guide? Please submit issues or pull requests on GitHub.

---

**Last Updated:** October 2025  
**Version:** 1.0
