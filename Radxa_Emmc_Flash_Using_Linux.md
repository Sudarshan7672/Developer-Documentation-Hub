# Flash Radxa Zero 3W eMMC using rkdeveloptool (Linux)

This guide explains how to flash the Woobly OS to the Radxa Zero 3W eMMC using rkdeveloptool on Linux.

## 🧰 Requirements

- Linux PC (Ubuntu/Debian recommended)
- Radxa Zero 3W board
- USB-A ↔ USB-C cable
- Board in Maskrom/Loader mode

---

## 1️⃣ Install rkdeveloptool (Linux)

```bash
sudo apt update
sudo apt install rkdeveloptool
```

Verify installation:

```bash
rkdeveloptool --version
```

---

## 2️⃣ Download RK3566 boot loader

Download:

```
https://dl.radxa.com/rock3/images/loader/rk356x_spl_loader_ddr1056_v1.12.109_no_check_todly.bin
```

Save as:

```
rk356x_spl_loader_ddr1056_v1.12.109_no_check_todly.bin
```

---

## 3️⃣ Download OS image (Woobly)

```
https://drive.google.com/file/d/1CP2R2Ron-aQa25aQiESUScF8QxoLTuyY/view?usp=sharing
```

After download:

```
Woobly_Master_OS_V1.img
```

---

## 4️⃣ Put Zero 3W in Maskrom / Loader mode

Connect board to PC using USB-C. Then check:

```bash
rkdeveloptool ld
```

Expected output:

```
DevNo=1  Vid=0x2207,Pid=0x350b  Maskrom
```

or

```
Loader
```

---

## 5️⃣ Load boot loader to device

```bash
sudo rkdeveloptool db rk356x_spl_loader_ddr1056_v1.12.109_no_check_todly.bin
```

Expected:

```
Downloading bootloader succeeded
```

---

## 6️⃣ Flash OS to eMMC

```bash
sudo rkdeveloptool wl 0 Woobly_Master_OS_V1.img
```

This writes the full image to eMMC starting at sector 0. Wait until completion:

```
Write LBA succeeded
```

---

## 7️⃣ Reboot device

```bash
sudo rkdeveloptool rd
```

Disconnect USB and power board normally. Woobly OS should boot from eMMC.

---

## ⚠️ Driver / permission issues (Linux)

If device not detected, check USB:

```bash
lsusb | grep 2207
```

If permission denied, create a udev rule:

```bash
sudo nano /etc/udev/rules.d/99-rockchip.rules
```

Add:

```
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```

Then reload:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Reconnect device.

---

## ✅ Complete flashing command sequence

```bash
rkdeveloptool ld
sudo rkdeveloptool db rk356x_spl_loader_ddr1056_v1.12.109_no_check_todly.bin
sudo rkdeveloptool wl 0 Woobly_Master_OS_V1.img
sudo rkdeveloptool rd
```

---

## 👍 Notes

- Works for Radxa Zero 3W (RK3566)
- Loader version `v1.12.109_no_check_todly` recommended by Radxa
- Image is written directly to eMMC
