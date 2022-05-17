---
layout: post
title:  "Steamlink"
date:   2020-05-23 18:00:00 -0500
categories: tech
tags: [godot, godot networking, game development]
comments: true
---

## Introduction

I recently got my hands on a Raspberry Pi 4B and installed the latest and greatest version of Raspberry Pi OS based on Debian 11 (bullseye).
I figured I'd try the 64-bit version of "Raspberry Pi OS Lite" since it was offered as a stable release.
According to [Valve's official setup guide](https://help.steampowered.com/en/faqs/view/6424-467A-31D9-C6CB)
for installing steamlink on a Raspberry Pi, it should be as easy as `sudo apt install steamlink`.

## The Problem
However, after installing and running `steamlink`, the app failed to load with errors about missing libraries.

```
shell: error while loading shared libraries: libbcm_host.so: cannot open shared object file: No such file or directory
screenblank: error while loading shared libraries: libbcm_host.so: cannot open shared object file: No such file or directory
```

## The Solution

I managed to sort through all the various libraries that were missing and created an install script.

Run this line on your Raspberry Pi to install all of steamlink's dependencies:
```

```
The script runs steamlink once in order to go through steamlink's first time setup which will prompt you to press Enter a few times.

You should now have a functional version of steamlink installed.
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
dtoverlay=vc4-fkms-v3d
```
TODO: add link to difference between KMS and FKMS


For 4K Televisions:
Steamlink doesn't support 4K at the moment, so the only way I was able to get this to work was to force the raspberry pi to boot using 1080p resolution on the HDMI output.
This will cause your TV to upscale to 4K to fill the full screen.

boot/config.txt
```
hdmi_group=1
hdmi_mode=16
```

Note about:
You are running with less than 128 MB video memory, you may need to go to the Raspberry Pi Configuration and increase your GPU memory.


Note about it (FKMS) working with retropie and kodi

## References

- [Godot Multiplayer API](https://docs.godotengine.org/en/stable/classes/class_multiplayerapi.html)
- [Godot Networking Tutorials](https://docs.godotengine.org/en/stable/tutorials/networking/index.html)
