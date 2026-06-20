# Android Boot Animations - Extracted from Device

**Device**: Android Head Unit (MediaTek-based)
**Date Extracted**: 2026-06-20
**Source**: ADB with root access

## Overview

This repository contains all boot-related graphics extracted from an Android device, including:
- Static boot logo (MediaTek LOGO partition)
- Standard boot animation
- XYAuto boot animation (actively used)
- Converted video files for easy viewing

---

## Directory Structure

```
android-boot-animations/
├── static-logo/           # Static boot logo from LOGO partition
│   ├── boot_logo.img      # Full partition image (3MB)
│   └── boot_splash.bmp    # Extracted BMP (1024x600, 24-bit)
│
├── standard-animation/    # Standard Android boot animation
│   ├── bootanimation.zip  # Original ZIP file (3.2MB)
│   └── frames/            # Extracted PNG frames
│       ├── desc.txt       # Animation descriptor (1024x600 @ 10fps)
│       ├── part0/         # 50 frames - plays once
│       └── part1/         # 30 frames - loops until boot
│
├── xyauto-animation/      # XYAuto boot animation (ACTIVE)
│   ├── bootanimation.zip  # Original ZIP file (6.5MB)
│   └── frames/            # Extracted JPG frames
│       ├── desc.txt       # Animation descriptor (1024x600 @ 5fps)
│       ├── part0/         # 44 frames - "Powered By android" animation
│       └── part1/         # 1 frame - static end frame
│
└── videos/                # MP4 conversions for easy viewing
    ├── bootanimation_part0.mp4        # Standard animation intro
    ├── bootanimation_part1.mp4        # Standard animation loop
    ├── bootanimation_full.mp4         # Standard animation complete
    ├── bootanimation_xyauto_part0.mp4 # XYAuto animation intro
    ├── bootanimation_xyauto_part1.mp4 # XYAuto static frame
    └── bootanimation_xyauto_full.mp4  # XYAuto animation complete
```

---

## Extraction Details

### 1. Static Logo (LOGO Partition)

**Partition**: `/dev/block/mmcblk0p14` → `/dev/block/platform/soc/11230000.mmc/by-name/LOGO`
**Size**: 3,145,728 bytes (3 MB)
**Format**: MediaTek logo partition format
- Header at offset 0x00 (512 bytes)
- BMP image at offset 0x200
- **Resolution**: 1024x600, 24-bit color

**Extraction Command**:
```bash
adb shell "su 0 dd if=/dev/block/mmcblk0p14 of=/sdcard/boot_logo.img"
adb pull /sdcard/boot_logo.img
dd if=boot_logo.img of=boot_splash.bmp bs=1 skip=512
```

### 2. Standard Boot Animation

**Location**: `/system/media/bootanimation.zip`
**Size**: 3,219,788 bytes (3.2 MB)
**Format**: Android boot animation ZIP

**Descriptor** (`desc.txt`):
```
1024 600 10
p 1 0 part0
p 0 0 part1
```
- Resolution: 1024x600
- Frame rate: 10 fps
- part0: plays once (50 frames, 5 seconds)
- part1: loops indefinitely (30 frames, 3 seconds)

**Extraction Command**:
```bash
adb pull /system/media/bootanimation.zip
unzip bootanimation.zip
```

### 3. XYAuto Boot Animation (ACTIVE)

**Location**: `/system/media/xyauto/bootanimation.zip`
**Size**: 6,451,364 bytes (6.5 MB)
**Format**: Android boot animation ZIP
**Status**: This is the animation actively displayed on boot

**Descriptor** (`desc.txt`):
```
1024 600 5
p 1 0 part0
p 0 0 part1
```
- Resolution: 1280x720 (higher than standard)
- Frame rate: 5 fps
- part0: animated intro (44 frames, 8.8 seconds)
- part1: static end frame (1 frame)

**Features**:
- High-resolution "Powered By android" animation
- Particle effects and glowing Android logo
- Professional branded appearance

**Extraction Command**:
```bash
adb pull /system/media/xyauto/bootanimation.zip
unzip bootanimation.zip
```

---

## Boot Sequence

1. **LOGO Partition** → Static BMP displayed by bootloader (very brief)
2. **XYAuto Animation** → Plays from `/system/media/xyauto/bootanimation.zip`
   - part0: 8.8 second animated intro
   - part1: static frame loops until Android fully loads

The system prioritizes `/system/media/xyauto/bootanimation.zip` over the standard location.

---

## Video Conversions

All animations have been converted to MP4 for easy viewing without Android tools:

**Standard Animation**:
- `bootanimation_full.mp4` - Complete sequence (13.7 seconds with 3 loops)

**XYAuto Animation**:
- `bootanimation_xyauto_full.mp4` - Complete sequence (8.8 seconds)

**FFmpeg Conversion Commands**:
```bash
# Standard part0 (1024x600 @ 10fps)
ffmpeg -framerate 10 -i part0/%02d.png -c:v libx264 -pix_fmt yuv420p bootanimation_part0.mp4

# XYAuto part0 (1280x720 @ 5fps)
ffmpeg -framerate 5 -pattern_type glob -i 'part0/*.jpg' -c:v libx264 -pix_fmt yuv420p bootanimation_xyauto_part0.mp4
```

---

## Device Information

**Architecture**: MediaTek-based Android head unit
**Platform**: `soc/11230000.mmc` (eMMC storage)
**Boot partition scheme**: Standard MediaTek layout with dedicated LOGO partition

---

## Notes

- The XYAuto animation is higher resolution (1280x720) than the standard animation (1024x600)
- Both animations coexist on the device but XYAuto takes priority
- The static LOGO partition BMP is rarely visible as Android boots quickly to the animated sequence
- All files extracted with root access via ADB

---

## Usage

To view the animations:
1. Open any MP4 file in the `videos/` directory
2. Or extract frames from ZIP files and view sequentially
3. To replace animations, push modified ZIP files back to device

To modify and push back to device:
```bash
# Push custom animation
adb push custom_bootanimation.zip /sdcard/
adb shell "su 0 mount -o remount,rw /system"
adb shell "su 0 cp /sdcard/custom_bootanimation.zip /system/media/xyauto/bootanimation.zip"
adb shell "su 0 chmod 644 /system/media/xyauto/bootanimation.zip"
adb reboot
```

---

**Repository created**: 2026-06-20
**Extracted by**: ADB root shell commands
