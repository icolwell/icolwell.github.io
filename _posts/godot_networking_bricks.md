## Establishing Connections

One game instance can connect to one or more other game instances.
Typically the server or host instance connects to all clients, but the clients each only connect to the server (Is this true?).

## Object Ownership (Network Master)

Instances of the game can control a node on a different instance.
For example the server instance can control the nodes on a client instance that represent all other players.
The single instance which is the controller of any node is called the "Network Master" of the node.

**Helper Functions**
`void set_network_master ( int id, bool recursive=true )` = Set the network master of a node and all its child nodes (recursive true by default).
`bool is_network_master ( ) const` = Returns true if the current instance is the network master of the node the function is called in.

## RPCs

Remote procedure calls can execute a function on another instance of the game.
The node scene path must be the same on both instances in order for the rpc to execute.

`rpc("function_name", <optional_args>)` = Executes the function on all connected instances.
`rpc_id(<instance_id>,"function_name", <optional_args>)` = Only executes the function on the instance specified by the instance_id.

`rset("variable", value)` = Updates the value of a member variable on all puppet nodes.
`rset_id(<instance_id>, "variable", value)` = Updates the value of a member variable only on the node specified by the instance_id

## Node Functions

RPCs can't just execute any function on another instance that have a matching node path.
This section explains how to define functions to allow them to be executed from other instances.

"master" = Instance that is the network master of the node.
"puppet" = Instance that is not the network master of the node.

`func` = Only executable from the same game instance.

`remote func` = Only executable from another connected instance of the game. Callable from any connected instance
`remotesync func` = Executes on all connected instances including the instance that called the function via RPC. Callable from all instances.

`master func` = Only executes on the instance that is the network master of the node. Only callable from puppets.
`mastersync func` = Executes on the instance that is the network master, and the instance that called the function via RPC. Only callable from puppets.

`puppet func` = Only executes on puppets. Only callable from masters.
`puppetsync func` = Executes on all connected puppets, and locally. Only callable from masters.

The same keywords apply to member variables

`var`

`remote var` = Only changed by another connected instance of the game. Changeable from any connected instance.
`remotesync var` = Changes both the local variable and all connected variables. Changeable from all instances.

`master var` = Only changes the variable on the network master. Only changeable from puppets.
`mastersync var` = Changes the variable on the network master and locally. Only changeable from puppets.

`puppet var` = Only changes the variable on puppet instances. Only changeable from masters.
`puppetsync var` = Changes the variable on puppet instances and locally. Only changeable from masters.


https://docs.godotengine.org/en/3.2/classes/class_multiplayerapi.html
