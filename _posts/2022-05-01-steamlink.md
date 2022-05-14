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

TODO: Add missing library errors here

## The Solution

I managed to sort through all the various libraries that were missing and created an install script.

Run this line on your Raspberry Pi to install all of steamlink's dependencies:
```

```

You should now have a functional version of steamlink installed.

However! there were still a couple of configs to tune
FKMS
force 1080p
snd-bcm2835.enable_compat_alsa=1


## References

- [Godot Multiplayer API](https://docs.godotengine.org/en/stable/classes/class_multiplayerapi.html)
- [Godot Networking Tutorials](https://docs.godotengine.org/en/stable/tutorials/networking/index.html)
