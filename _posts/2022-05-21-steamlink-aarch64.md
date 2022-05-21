---
layout: post
title:  "Install Steamlink on 64-bit Raspberry Pi OS Bullseye"
date:   2022-05-21 8:00:00 -0400
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

I managed to sort through all the various libraries that were missing and created an install script.

Run this line on your Raspberry Pi to install all of steamlink's dependencies:
```
curl -sSL https://raw.githubusercontent.com/icolwell/install_scripts/master/steamlink_install.bash | bash
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
