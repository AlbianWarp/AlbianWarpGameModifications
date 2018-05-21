Albian Warp CAOS Developer Documentation
========================================

So, you're a CAOS Developer who wants to:

* Improve Albian Warp
* Create agents that can 'talk' to each other through the Albian Warp functionality
* or just understand how all this works

This Documentation is for you!


##Contents

* Direct Messaging Agents (DMAs)
	* Sending Mail
	* Sending Creatures
* Real-Time Direct Messaging Agents (RTDMAs)
	* Chat Invites and Notifcations
	* Chat Messages
* Writing your Own
	* About net: command replacements

## Direct Messaging Agents (DMAs)

Most communication done through Albian Warp is made possible by Direct Messaging Agents. These are simply agents that are created with the classifier `1 1 35753` and are set with the NAME variable "aw_recipient" containing the username of the person that the message will be sent to. They contain no scripts, but they can contain any other NAME variables that the recieving scripts need to process the message.

When an outgoing DMA is created, the client picks it up, converts it to a JSON format, and sends it to the server. The server then sends it to the client of the person it is intended to recieve. The client turns the message into an agent and injects it into the recieving game. This agent is classified as `1 1 35754` containing the variables "aw\_sender" containing the username of the sender, and "aw\_date" containing the server date and time it was sent. This in the same format as is returned by the CAOS command `RTIM` and can be parsed with `RTIF`. It also carries over any NAME variables that were defined on that agent by the sender.