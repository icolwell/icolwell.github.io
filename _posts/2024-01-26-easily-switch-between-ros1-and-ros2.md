---
layout: post
title:  "Easily Switch Between ROS1 and ROS2 Terminal Environments"
date:   2024-01-26 18:00:00 -0500
categories: tech
tags: [ros1, ros2, terminal, environment, robot operating system]
comments: true
---

## Introduction

It's 2024 so maybe this post is a bit late, but hopefully it's helpful to anyone that needs to run both ROS1 and ROS2 on the same Ubuntu system.
BTW, if you are in the process of migrating from ROS1 to ROS2, I highly recommend the [ros1_bridge](https://index.ros.org/p/ros1_bridge/) which allows you to run nodes from both ROS1 and ROS2 at the same time.

## Background

If you are familiar with ROS, you know that you need to source the ROS setup scripts into your terminal environment in order to use `ros*` commands.

For ROS1 you'd do something like this:
```bash
source /opt/ros/noetic/setup.bash
```
For ROS2 you'd do something like this:
```bash
source /opt/ros/foxy/setup.bash
```

Those lines are typically added to a user's `.bashrc` file.

But what if you started a terminal with ROS1 sourced and then want to switch and use ROS2 commands?
You could:
- Close the terminal and start a fresh one ... but my Tmux panes are all setup nice!
- Clear all the ROS1-related environment variables and re-source the ROS2 setup scripts ... sounds tedious. What if you have multiple ROS1 or ROS2 dev workspaces that you need to source and/or extend?

It starts getting messy and annoying to switch environments.

## Solution

The solution is the `rs` command! 
Maybe it's short for "ros", or maybe it means "ros switch". 
Whatever is easier to remember.

### Installation

All you need to do is download [this gist](https://gist.github.com/icolwell/e05d8f62f66cf82e862346c655b55a98) by running the following commands:

```
cd ~
curl -sSLO https://gist.githubusercontent.com/icolwell/e05d8f62f66cf82e862346c655b55a98/raw/edaf6c2e391c6b92eaa05acc81e080c7bb1f3fa4/rs.bash
```

Then add the following lines to your `.bashrc` file:

```bash
# Add the `rs` command
source ~/rs.bash
# Choose ROS1 by default
rs 1
```

The above two lines will give you the ability to run the `rs` command from any terminal and start the terminal with a ROS environment sourced by default (ROS1 in the above example).

### Usage

Open a new terminal, notice that it shows:
```
ROS 1 Environment Sourced
```

If you want to switch to ROS2 simply type:
```
rs 2
```

And now it says 
```
ROS 2 Environment Sourced
```

Nice!


### Customization

The `rs.bash` script contains two sections, one for each version of ROS. 
Edit those sections as-needed to source whatever other workspaces you may have.


## Related Links

- [How to Switch Between ROS 1 and ROS 2 in Ubuntu Linux](https://automaticaddison.com/how-to-switch-between-ros-1-and-ros-2-in-ubuntu-linux/)
- [\[ROS2\] Sourcing colcon-generated setup.bash includes ROS1 catkin workspace](https://answers.ros.org/question/355967/ros2-sourcing-colcon-generated-setupbash-includes-ros1-catkin-workspace/)
