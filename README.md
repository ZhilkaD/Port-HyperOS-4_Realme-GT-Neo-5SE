# Port HyperOS 4 for Realme GT Neo 5 SE (senna_jr / RMX3700)

![Android](https://img.shields.io/badge/Android-17%20(SDK%2037)-blue?style=flat-square)
![Status](https://img.shields.io/badge/Build-UNOFFICIAL-orange?style=flat-square)
![SELinux](https://img.shields.io/badge/SELinux-Permissive-yellow?style=flat-square)

Unofficial port of **HyperOS 4 (Android 17)** for the **Realme GT Neo 5 SE** (`senna_jr` / `RMX3700`).
HyperOS 4 system half (Xiaomi 15 `dada` dump `OS4.0.0.7.XOCMIXM`) running on top of the stock RealmeUI vendor stack (SM7475 Snapdragon 7+ Gen 2).
*Personal port, not affiliated with Xiaomi / realme.*

* **System:** `CP2A.260605.016`, SDK 37 (Android 17)
* **Vendor:** stock RealmeUI `senna_jr` vendor stack, kernel `5.10` Ukee/Senna
* **Identity:** base reports as Xiaomi 15 (`PJZ110` / `dada`) on realme hardware

> **Important Notices:**
> * **Experimental port:** HyperOS 4 / Android 17 was never officially released for this device. Expect minor rough edges.
> * **SELinux permissive** (patched bootloader cmdline). adb runs with root access. Do not use as a daily driver for banking or sensitive confidential data.
> * **Integrity:** AVB is off (`vbmeta` flashed with verity and verification disabled). Play Integrity will fail by default without Magisk / KernelSU modules.
> * **No GMS:** Clean HyperOS product partition without Google apps preinstalled. Gboard, MicroG or NikGapps can be sideloaded.
> * **Super partition layout:** `my_stock` is replaced by a 4 KB dummy image to free ~5 GB inside `super`, allowing the large HyperOS `product` (3.8 GB) to fit alongside Realme's `my_*` partitions.
> * **Single-slot:** The ROM occupies slot A only. Slot B logical partitions are deleted to maximize free space in the `super` partition.
> * **No OTA:** All updates are manual fastboot / fastbootd flashes.
> * **First boot takes 5–10 minutes** — do not interrupt or reboot the phone.

---

## Changelog — 2026-10-15 (v1.0 Initial Release)

* Initial public HyperOS 4 (Android 17) port for Realme GT Neo 5 SE.
* Converted all system images to raw EROFS / ext4 for full **Fastboot Firmware Flasher (FFF)** compatibility (eliminates `Invalid argument` chunking errors).
* Fixed `Not enough space to resize partition` bug by shrinking `my_stock` to 4 KB first, freeing 4.96 GB in `super`.
* Preserved all 10 Realme vendor `my_*` partitions (`my_product`, `my_manifest`, `my_region`, etc.) to prevent vendor init mount aborts and fastboot loops.
* Smooth 144 Hz display refresh rate, SurfaceFlinger, and HyperOS UI animations fully operational.
* Mobile network, VoLTE, calls, SMS, Wi-Fi (2.4/5 GHz), and Bluetooth working out of the box.

---

## Tested Base (Read Before Flashing)

Tested on **Realme GT Neo 5 SE** (`RMX3700` / `senna_jr`):
```text
ro.build.display.id = RMX3700 / RMX3701 (RealmeUI 5.0 / 6.0 base)
ro.boot.prjname     = 22623
```

Check your project parameters before flashing:
```bash
adb shell getprop ro.boot.prjname
```

---

## Hardware Status

### Working
* Boot and system stability across reboots
* Display at 144 Hz with smooth SurfaceFlinger rendering
* Touch responsiveness
* Speaker, earpiece, stereo sound, microphone
* Cellular network, mobile data (4G/5G), voice calls, SMS
* Wi-Fi (dual-band 2.4/5 GHz), Bluetooth
* Camera hardware (all sensors), video recording
* Basic sensors (accelerometer, compass, proximity)
* Fast battery charging

### Known Issues
* **In-display fingerprint (UDFPS)** — Goodix/Oplus vendor HAL requires adaptation for Android 17 framework interfaces.
* **Auto-brightness** — Ambient light sensor auto-adjustment is inconsistent; manual slider works fine.
* **No GMS** — Google Play Services are not preinstalled; install MicroG or sideload apps manually.
* Minor cosmetic background service warnings in logcat.

---

## Installation Guide

### Prerequisites
* Unlocked bootloader.
* [platform-tools](https://developer.android.com/studio/releases/platform-tools) installed and added to PATH (or located in `C:\platform-tools`).
* **Full backup:** all data on the internal storage will be erased.

---

### Option 1: Fastboot Firmware Flasher (Recommended)

1. Open **Fastboot Firmware Flasher (FFF)**.
2. Select the `HyperOS4_Port_RMX3700_v1.0.zip` package (or the unzipped folder).
3. Power off the phone -> hold **Volume Down + Power** to enter Fastboot mode.
4. Connect the USB cable to the PC and click **Flash**.

---

### Option 2: flash.bat (One-Click)

1. Extract the release archive to a short path without spaces or non-ASCII characters (e.g. `C:\senna-flash\HyperOS4_Port_RMX3700_v1.0`).
2. Power off -> hold **Volume Down + Power** (bootloader mode).
3. Connect the phone to the PC via USB.
4. Run `flash.bat` as Administrator.
5. The script will flash the bootloader images, transition to `fastbootd`, allocate `super`, flash all partitions, erase userdata, and reboot.
6. First boot takes 5–10 minutes.

---

### Option 3: Manual Commands

**Phase 1: Bootloader stage**
```cmd
set PATH=C:\platform-tools;%PATH%
fastboot set_active a
fastboot --disable-verity --disable-verification flash vbmeta BOOTLOADER\vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system BOOTLOADER\vbmeta_system.img
fastboot flash vbmeta_vendor BOOTLOADER\vbmeta_vendor.img
fastboot flash boot_a BOOTLOADER\boot.img
fastboot flash dtbo_a BOOTLOADER\dtbo.img
fastboot flash vendor_boot_a BOOTLOADER\vendor_boot.img
fastboot flash recovery_a BOOTLOADER\recovery.img
fastboot reboot fastboot
```

**Phase 2: fastbootd stage (super partitions)**
```cmd
:: Free space in super by shrinking my_stock first
fastboot flash my_stock_a SYSTEM\my_stock.img

:: Clean and recreate product
fastboot delete-logical-partition product_a
fastboot create-logical-partition product_a 0
fastboot flash product_a SYSTEM\product.img

:: Flash remaining system and vendor partitions
fastboot flash system_a SYSTEM\system.img
fastboot flash system_ext_a SYSTEM\system_ext.img
fastboot flash vendor_a SYSTEM\vendor.img
fastboot flash vendor_dlkm_a SYSTEM\vendor_dlkm.img
fastboot flash odm_a SYSTEM\odm.img
fastboot flash odm_dlkm_a SYSTEM\odm_dlkm.img
fastboot flash my_product_a SYSTEM\my_product.img
fastboot flash my_manifest_a SYSTEM\my_manifest.img
fastboot flash my_preload_a SYSTEM\my_preload.img
fastboot flash my_carrier_a SYSTEM\my_carrier.img
fastboot flash my_region_a SYSTEM\my_region.img
fastboot flash my_company_a SYSTEM\my_company.img
fastboot flash my_engineering_a SYSTEM\my_engineering.img
fastboot flash my_heytap_a SYSTEM\my_heytap.img
fastboot flash my_bigball_a SYSTEM\my_bigball.img

:: Wipe and reboot
fastboot erase metadata
fastboot erase userdata
fastboot set_active a
fastboot reboot
```

---

## Download

* **Release Package (7.05 GB):** [Download on Google Drive](https://drive.google.com/file/d/1jIPY-VSC3NWZ5IgdhIYuprlX-Vbpvbcm/view)
* Checksums: verified SHA-256 included in `sha256.txt`.
