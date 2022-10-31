---
layout: post
title:  "Install Steamlink on 64-bit Raspberry Pi OS Bullseye"
date:   2022-05-21 8:00:00 -0400
updated: 2022-10-30 22:00:00 -0400
categories: tech
tags: [raspberrypi, steamlink, aarch64, bullseye]
comments: true
---

## Introduction

I recently got my hands on a Raspberry Pi 4B and installed the latest and greatest version of Raspberry Pi OS based on Debian 11 (bullseye).
I figured I'd try the 64-bit version of "Raspberry Pi OS Lite" since it was offered as a stable release.
According to [Valve's official setup guide](https://help.steampowered.com/en/faqs/view/6424-467A-31D9-C6CB)
for installing steamlink on a Raspberry Pi, it should be as easy as `sudo apt install steamlink`.

## The Problem

However, after installing and running `steamlink`, the app failed to start with errors about missing libraries:

```
shell: error while loading shared libraries: libbcm_host.so: cannot open shared object file: No such file or directory
screenblank: error while loading shared libraries: libbcm_host.so: cannot open shared object file: No such file or directory
```

## The Solution

Before proceeding, it is important to understand which version of 64-bit Raspberry Pi OS you have installed.
There are two versions:
- **Raspberry Pi OS Lite**: No GUI, slimmed-down OS that boots to a terminal login prompt.
- **Raspberry Pi OS with desktop**: Full desktop GUI that boots to a login GUI screen.

If you have "Raspberry Pi OS Lite", you can go ahead and skip to the "Installing Steamlink" section below.
If you have "Raspberry Pi OS with desktop", then you need to exit the X server (the GUI) with the following keybaord shortcuts:

- `Ctrl-Alt-F2`: Exit desktop GUI and open the TTY2 terminal.
- `Ctrl-Alt-F7`: Return to the desktop GUI.

If you try running `steamlink` from a terminal window in the desktop GUI, you will get the following error:

![Steamlink X11 Error](/assets/steamlink_x11_error.png)

So make sure you follow the rest of this guide in TTY2.
Also, once `steamlink` is fully installed, you will always need to run it from a TTY outside the GUI.

### Installing Steamlink

I managed to sort through all the various libraries that were missing and created an install script.
Run this line on your Raspberry Pi to install all of steamlink's dependencies:
```
curl -sSL https://raw.githubusercontent.com/icolwell/install_scripts/master/steamlink_install.bash | bash
```
Another option (especially if you are reading this guide on your Raspberry Pi browser) is to first download the script, then execute the script once switched to TTY2.
For example:
```
curl -sSL https://raw.githubusercontent.com/icolwell/install_scripts/master/steamlink_install.bash --output steamlink_install.bash
# "Ctrl-Alt-F2" and login
bash steamlink_install.bash
```

The script runs steamlink once in order to go through steamlink's first time setup which will prompt you to press Enter a few times.
Once the script completes, you should now have a functional version of steamlink installed.

However! Launching steamlink will appear to work, but once you start streaming a game, you may get a black screen.
There are a few small things to adjust in the boot config in order for steamlink to be able to decode the video stream.

In the `/boot/config.txt` file you should see the following:

```
# Enable DRM VC4 V3D driver
dtoverlay=vc4-kms-v3d
```
The default is to use the KMS driver, but steamlink currently only seems to work with the FKMS driver.
Change the line to the following by simply adding an "f":
```
# Enable DRM VC4 V3D driver
dtoverlay=vc4-fkms-v3d
```

I suggest googling ["FKMS vs. KMS driver"](https://www.google.com/search?q=FKMS+vs.+KMS+driver)
if you want to learn more about the differences between KMS and FKMS.

### [Optional] Suppress warning about video memory
When launching steamlink, I get this warning:
```
You are running with less than 128 MB video memory, you may need to go to the Raspberry Pi Configuration and increase your GPU memory.
```
However, everything seems to work fine, and I believe it's safe to ignore.
That being said, if you'd like to get rid of the warning, increase your video memory by using `raspi-config` or adding the following line to `/boot/config.txt`:
```
gpu_mem=128
```

### [Optional] For 4K televisions:
Steamlink doesn't support 4K at the moment, so the only way I was able to get this to work was to force the raspberry pi to boot using 1080p resolution on the HDMI output.
This will cause your TV to upscale to 4K to fill the full screen.

Edit the `/boot/config.txt` file to add the following lines:
```
hdmi_group=1
hdmi_mode=16
```

## Conclusion

I was nervous about switching to FKMS driver since it seems like KMS is the recommended driver for bullseye and future versions.
However, I have been able to run steamlink, kodi, and some retropie emulators just fine with FKMS on 64-bit bullseye.
Hopefully steamlink will officially support the KMS driver in the future.

I hope this guide helped save you some time!
Let me know if there's something I can improve or if steamlink has been properly released on 64-bit bullseye such that this script is no longer needed.

## References

- [Valve's official steamlink setup guide](https://help.steampowered.com/en/faqs/view/6424-467A-31D9-C6CB)
