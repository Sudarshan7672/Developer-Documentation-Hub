# Radxa Zero 3 UART2 Configuration Guide

Complete step-by-step guide to enable and use UART2 on Radxa Zero 3 (RK3566)
Os Image Link - https://www.armbian.com/radxa-zero-3/

---

## Overview

By default, UART2 on Radxa Zero 3 is disabled or used by the kernel debug console (fiq-debugger). This guide shows how to properly enable it for user applications.

---

## Hardware Specifications

- **Chip**: Rockchip RK3566
- **UART2 Device**: `/dev/ttyS2`
- **Memory Address**: `0xfe660000`
- **Pin Configuration**: uart2m0-xfer mode

### Pin Mapping (40-pin Header)

| Pin Number | Function | GPIO Name |
|------------|----------|-----------|
| Pin 8      | UART2 TX | GPIO0-25  |
| Pin 10     | UART2 RX | GPIO0-24  |
| Pin 6/9/14/20 | GND   | Ground    |

---

## Step 1: Check Current Status

### 1.1 Check Available Serial Ports
```bash
ls -l /dev/ttyS*
```

**Expected (before fix)**: Only `/dev/ttyS1` exists  
**Expected (after fix)**: Both `/dev/ttyS1` and `/dev/ttyS2` exist

### 1.2 Check Device Tree Status
```bash
cat /sys/firmware/devicetree/base/serial@fe660000/status
```

**Before fix**: `disabled`  
**After fix**: `okay`

### 1.3 Check Kernel Messages
```bash
dmesg | grep ttyS
```

**Before fix**: Only ttyS1 appears  
**After fix**: Both ttyS1 and ttyS2 appear

### 1.4 Check What's Using UART Ports
```bash
# Check if any service is using the ports
for port in /dev/ttyS*; do
    echo -n "$port: "
    fuser $port 2>/dev/null || echo "FREE"
done

# Check processes using UART
lsof | grep ttyS
```

---

## Step 2: Locate Device Tree Overlays

### 2.1 Find Overlay Directory
```bash
cd /boot/dtb/rockchip/overlay
ls -la | grep uart
```

### 2.2 Verify UART2 Overlay Exists
```bash
ls -la rk3568-uart2-m0.dtbo
```

**Expected output**:
```
-rwxr-xr-x 1 root root 818 Oct 16 09:06 rk3568-uart2-m0.dtbo
```

---

## Step 3: Configure Boot Parameters

### 3.1 Backup Current Configuration
```bash
sudo cp /boot/armbianEnv.txt /boot/armbianEnv.txt.backup
```

### 3.2 Edit Boot Configuration
```bash
sudo nano /boot/armbianEnv.txt
```

### 3.3 Update Configuration

**Change these lines:**

**BEFORE:**
```ini
console=both
overlay_prefix=rk35xx
overlays=uart1 uart2 uart3 uart4
extraargs=cma=256M
```

**AFTER:**
```ini
console=serial
overlay_prefix=rk3568
overlays=uart2-m0
extraargs=cma=256M
```

### Complete Configuration Example:
```ini
verbosity=1
bootlogo=false
console=serial
extraargs=cma=256M
overlay_prefix=rk3568
fdtfile=rockchip/rk3566-radxa-zero3.dtb
rootdev=UUID=b736f7c7-11a7-483d-b224-e062f6aab6e7
rootfstype=ext4
overlays=uart2-m0
usbstoragequirks=0x2537:0x1066:u,0x2537:0x1068:u
```

### Key Changes Explained:

1. **`console=serial`**: Changes from `both` to prevent UART2 being claimed by debug console
2. **`overlay_prefix=rk3568`**: Changes from `rk35xx` to match actual overlay file names
3. **`overlays=uart2-m0`**: Enables only UART2 in m0 mode (simplified from listing all UARTs)

---

## Step 4: Reboot and Verify

### 4.1 Reboot System
```bash
sudo reboot
```

### 4.2 Verify UART2 is Available
```bash
# Check device files
ls -l /dev/ttyS*

# Expected output:
# crw-rw---- 1 root dialout 4, 65 Oct 22 19:55 /dev/ttyS1
# crw-rw---- 1 root dialout 4, 66 Oct 22 19:55 /dev/ttyS2
```

### 4.3 Check Kernel Recognition
```bash
dmesg | grep ttyS

# Expected output:
# [   18.894716] fe650000.serial: ttyS1 at MMIO 0xfe650000 (irq = 30, base_baud = 1500000) is a 16550A
# [   18.896087] fe660000.serial: ttyS2 at MMIO 0xfe660000 (irq = 31, base_baud = 1500000) is a 16550A
```

### 4.4 Verify Device Tree Status
```bash
cat /sys/firmware/devicetree/base/serial@fe660000/status

# Expected output: okay
```

### 4.5 Check Kernel Command Line
```bash
cat /proc/cmdline | grep console

# Should NOT contain: console=ttyS2
# Should contain: console=tty1
```

---

## Step 5: Test UART2 Loopback

### 5.1 Hardware Setup
Connect **Pin 8 to Pin 10** with a jumper wire (TX to RX loopback)

### 5.2 Create Test Script
```bash
nano uart_test.py
```

```python
#!/usr/bin/env python3
import serial
import time

port = '/dev/ttyS2'
baudrate = 115200

try:
    ser = serial.Serial(
        port=port,
        baudrate=baudrate,
        timeout=1,
        rtscts=False,
        dsrdtr=False,
        xonxoff=False
    )
    
    print(f"✓ Opened {port} at {baudrate} baud")
    
    # Clear buffers
    ser.reset_input_buffer()
    ser.reset_output_buffer()
    time.sleep(0.1)
    
    # Send test data
    test_data = b'Hello UART2!\n'
    print(f"Sending: {test_data}")
    ser.write(test_data)
    ser.flush()
    
    # Wait and read
    time.sleep(0.5)
    
    if ser.in_waiting > 0:
        received = ser.read(ser.in_waiting)
        print(f"✓ Received: {received}")
        
        if received == test_data:
            print("✓✓✓ SUCCESS! UART2 loopback working!")
        else:
            print("⚠ Data mismatch!")
    else:
        print("✗ No data received")
    
    ser.close()
    
except Exception as e:
    print(f"✗ Error: {e}")
```

### 5.3 Run Test
```bash
python3 uart_test.py
```

**Expected output:**
```
✓ Opened /dev/ttyS2 at 115200 baud
Sending: b'Hello UART2!\n'
✓ Received: b'Hello UART2!\n'
✓✓✓ SUCCESS! UART2 loopback working!
```

---

## Troubleshooting

### Issue 1: UART2 Still Not Appearing

**Check overlay is actually applied:**
```bash
cat /sys/firmware/devicetree/base/serial@fe660000/status
```

If still `disabled`, check:
```bash
# Verify overlay file exists
ls -la /boot/dtb/rockchip/overlay/rk3568-uart2-m0.dtbo

# Check boot configuration syntax
cat /boot/armbianEnv.txt
```

### Issue 2: Permission Denied

**Add user to dialout group:**
```bash
sudo usermod -aG dialout $USER
# Log out and back in
```

### Issue 3: Device Busy

**Check what's using the port:**
```bash
lsof | grep ttyS2
fuser /dev/ttyS2

# If something is using it, kill the process:
sudo kill <PID>
```

### Issue 4: Console Still on ttyS2

**Verify kernel command line:**
```bash
cat /proc/cmdline
```

Should NOT contain `console=ttyS2`. If it does:
- Double-check `/boot/armbianEnv.txt`
- Ensure `console=serial` or `console=tty1` is set
- Reboot again

---

## Common Issues Summary

| Issue | Cause | Solution |
|-------|-------|----------|
| `/dev/ttyS2` missing | Wrong overlay prefix | Change `overlay_prefix` to `rk3568` |
| Device tree disabled | Overlay not applied | Use `overlays=uart2-m0` |
| Port busy | Debug console using it | Set `console=serial` |
| No loopback data | No physical connection | Connect Pin 8 to Pin 10 |
| Permission denied | Not in dialout group | `sudo usermod -aG dialout $USER` |

---

## Additional Configuration Options

### Enable Multiple UARTs

To enable other UARTs along with UART2:
```ini
overlays=uart2-m0 uart3-m0 uart4-m1
```

### Available UART Overlays (RK3568)

```bash
ls /boot/dtb/rockchip/overlay/rk3568-uart*
```

- `rk3568-uart0.dtbo`
- `rk3568-uart1-m1-full.dtbo`
- `rk3568-uart2-m0.dtbo` ← Used in this guide
- `rk3568-uart3-m0.dtbo`
- `rk3568-uart3-m1.dtbo`
- `rk3568-uart4-m1.dtbo`
- `rk3568-uart5-m0-full.dtbo`
- `rk3568-uart5-m1.dtbo`
- `rk3568-uart7-m1.dtbo`
- `rk3568-uart7-m2.dtbo`
- `rk3568-uart8-m1.dtbo`
- `rk3568-uart9-m1.dtbo`

---

## Python Serial Configuration

### Basic Setup
```python
import serial

ser = serial.Serial(
    port='/dev/ttyS2',
    baudrate=115200,
    bytesize=serial.EIGHTBITS,
    parity=serial.PARITY_NONE,
    stopbits=serial.STOPBITS_ONE,
    timeout=1,
    rtscts=False,      # No hardware flow control
    dsrdtr=False,      # No DSR/DTR
    xonxoff=False      # No software flow control
)
```

### Install PySerial
```bash
sudo apt-get update
sudo apt-get install python3-serial
```

---

## Summary Checklist

- [ ] Backup `/boot/armbianEnv.txt`
- [ ] Change `overlay_prefix=rk3568`
- [ ] Set `overlays=uart2-m0`
- [ ] Set `console=serial` (not `both`)
- [ ] Reboot system
- [ ] Verify `/dev/ttyS2` exists
- [ ] Check device tree status is `okay`
- [ ] Test with loopback (Pin 8 → Pin 10)
- [ ] Add user to `dialout` group if needed

---

## Reference

- **Device**: Radxa Zero 3
- **SoC**: Rockchip RK3566
- **UART2 Address**: 0xfe660000
- **Pins**: Pin 8 (TX/GPIO0-25), Pin 10 (RX/GPIO0-24)
- **Device**: /dev/ttyS2
- **Overlay**: rk3568-uart2-m0.dtbo

---

**Document Version**: 1.0  
**Last Updated**: October 2025  
**Status**: Verified Working
