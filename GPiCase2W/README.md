# Batocera Fix for Retroflag GPi Case 2W (640x480 resolution)

> [!NOTE]
> This repository provides a definitive fix for Batocera (v39+) display issues on the Retroflag GPi Case 2W using a Raspberry Pi Zero 2W.

## The Problem
By default, Batocera (bcm2836 image) includes a legacy script that detects the Pi Zero 2W and mistakenly applies GPi Case 1 settings (320x240). This results in a distorted, "psychedelic" green/blue image or a complete failure to initialize the display driver.

## Installation
### 1. Neutralize the Auto-Install Script
The script that overwrites the configuration at every boot must be disabled. **Connect via SSH** and run:
```
# Remount the system as Read-Write
mount -o remount,rw /
# Empty the script content to stop it from running
true > /usr/bin/batocera-gpicase-install
# Remove execution rights
chmod -x /usr/bin/batocera-gpicase-install
# Save the changes to the system overlay
batocera-save-overlay
```

### 2. Configure config.txt

Edit **/boot/config.txt* and replace its content with the following settings. These force the ***fKMS*** driver and the correct ***640x480*** DPI timings.
```
[all]
# Keep the GPU active for the splash screen
disable_splash=0
disable_overscan=1

# Driver: fKMS is required for display initialization on v39+
dtoverlay=vc4-fkms-v3d

# Framebuffer Lock
framebuffer_ignore_alpha=1
framebuffer_priority=2
framebuffer_width=640
framebuffer_height=480

# DPI (Parallel Display Interface) Configuration
dtoverlay=dpi24
enable_dpi_lcd=1
display_default_lcd=1
dpi_group=2
dpi_mode=87

# RGB Order: 0x6f006 fixes inverted colors (Red appearing as Blue)
dpi_output_format=0x6f006

# Precise 640x480 @ 60Hz Timings
hdmi_timings=640 0 16 4 8 480 0 4 2 4 0 0 0 60 0 19200000 1

# Audio Fix (PWM 2-Channel)
dtparam=audio=on
dtoverlay=pwm-2chan,pin=18,func=2,pin2=19,func2=2
```
_full config.txt attached_

### 3. Global System Settings

To ensure stability, **batocera-boot.conf** (in the boot partition) and **batocera.conf** (in /userdata/system/) must be updated:

In batocera-boot.conf:
```
case=none
```

In batocera.conf:
```
system.wayland=0
display.rotate=0
```
_full batocera-boot.conf and batocera.conf attached_
