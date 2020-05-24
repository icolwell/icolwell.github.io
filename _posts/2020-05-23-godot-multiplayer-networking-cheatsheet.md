---
layout: post
title:  "Godot Multiplayer Networking Cheatsheet"
date:   2020-05-23 18:00:00 -0500
categories: tech
tags: [godot, godot networking, game development]
comments: true
---

## Introduction

This page is meant to be a quick reference guide to facilitate finding functions or specific syntax required for Godot multiplayer networking.
I suggest also reading the [official docs](https://docs.godotengine.org/en/stable/tutorials/networking/index.html), some of the content in this post is directly copied from the docs.

## Establishing Connections

One game instance can connect to one or more other game instances.
The other game instance can be running on the same computer or a different computer.
In general, there are two typical connection strategies that video games use:
1. **Client-Server**  
In this case, each player's game instance connects to one central game server instance.
The server instance is usually a special version of the game that doesn't have a GUI and is running at all times waiting for players to connect.
There is no need for players to connect to each other, they only need to establish a connection to the server.

2. **Peer-to-Peer**  
In the peer-to-peer case, any player can activate their game instance as a server (aka host) and allow other players to connect to them.
The host game instance is only active as long as the hosting player keeps their game open.
The hosting player can participate in the game at the same time as hosting it.
Each player still only connects to the game host, but the game host needs to connect to all other players.

### Creating a server
```
var peer = NetworkedMultiplayerENet.new()
peer.create_server(SERVER_PORT, MAX_PLAYERS)
get_tree().network_peer = peer
```

### Connecting to a server as a client
```
var peer = NetworkedMultiplayerENet.new()
peer.create_client(SERVER_IP, SERVER_PORT)
get_tree().network_peer = peer
```
If you only want to test connections locally at first, use `127.0.0.1` as the `SERVER_IP`.

### Connection-related scene tree signals
Each game instance has a unique automatically-generated `network peer id`.
The server instance always has an id of 1 while connected clients have a random 9-digit integer.

`connected_to_server()`  
Only emitted on clients.  

`connection_failed()`  
Only emitted on clients.  

`network_peer_connected(int id)`  
Emitted on all game instances when a client connects to the server.
`id` is the `network peer id`.  

`network_peer_disconnected(int id)`  
Emitted on all game instances when a client disconnects.  

`server_disconnected()`  
Only emitted on clients.

## Node Ownership (Network Master)

An Instance of the game can act as the "steward" or "master" of a node on a different instance.
For example the server instance can govern the nodes on a client instance that represent all other players.
The single instance which is the master of any node is called the "Network Master" of the node.

Set node ownership using the following functions:

`void set_network_master(int id, bool recursive=true)`  
Set the network master of a node and all its child nodes (recursive true by default).  

`bool is_network_master() const`  
Returns true if the current game instance is the network master of the node the function is called in.

**Note:** Assigning node ownership on its own doesn't do anything.
Pairing node ownership with RPCs and variable synchronization is where it becomes useful.

## Remote Procedure Call (RPC)

RPCs can execute functions on another instance of the game.
The node scene path must be the same on both instances in order for the rpc to execute.

`rpc("function_name", <optional_args>)`  
Executes the function on all connected instances.  
`rpc_id(<instance_id>, "function_name", <optional_args>)`  
Only executes the function on the instance specified by the instance_id.

### Defining Node Functions

RPCs can't just execute any function on another instance that has a matching node path.
This section explains how to define functions to allow them to be executed from other instances.

`master` = Node which is the network master.  
`puppet` = Node which is not the network master.  

`func`  
Only executable from the same game instance.  

`remote func`  
Only executable from another connected instance of the game.
Callable from any connected instance.  
`remotesync func`  
Executes on all connected instances including the instance that called the function via RPC.
Callable from all instances.  

`master func`  
Only executes on the instance that is the network master of the node.
Only callable from puppets.  
`mastersync func`  
Executes on the instance that is the network master, and the instance that called the function via RPC.
Only callable from puppets.  

`puppet func`  
Only executes on puppets.
Only callable from masters.  
`puppetsync func`  
Executes on all connected puppets, and locally.
Only callable from masters.

## Remote Variable Synchronization

Like functions, variables can also be updated remotely.

`rset("variable", value)`  
Updates the value of a member variable locally and remotely.  
`rset_id(<instance_id>, "variable", value)`  
Updates the value of a member variable only on the node specified by the instance_id.

### Defining Node Variables

The same keywords apply to member variables.

`var`  
Regular variable, can not be changed remotely.

`remote var`  
Only changed by another connected instance of the game.
Changeable from any connected instance.  
`remotesync var`  
Changes both the local variable and all connected variables.
Changeable from any connected instance.  

`master var`  
Only changes the variable on the network master.
Only changeable from puppets.  
`mastersync var`  
Changes the variable on the network master and locally.
Only changeable from puppets.

`puppet var`  
Only changes the variable on puppet instances.
Only changeable from masters.  
`puppetsync var`  
Changes the variable on puppet instances and locally.
Only changeable from masters.

## References

- [Godot Multiplayer API](https://docs.godotengine.org/en/stable/classes/class_multiplayerapi.html)
- [Godot Networking Tutorials](https://docs.godotengine.org/en/stable/tutorials/networking/index.html)
