Albian Warp CAOS Developer Documentation
========================================

So, you're a CAOS Developer who wants to:

* Improve Albian Warp
* Create agents that can 'talk' to each other through the Albian Warp functionality
* or just understand how all this works

This Documentation is for you!


## Contents

* Contact Formatting
* NET: Command Replacement CAOS Functions
* The Who's Wanted Registers
* Direct Messaging Agents (DMAs)
	* Sending Mail
* Real-Time Direct Messaging Agents (RTDMAs)
	* Chatting
		* REQU Messages
		* CHAT Messages
* Send Creature Agents (SCAs)
* Writing your Own
	* Other GAME and EAME Variables
	* Other CAOS Functions

## Contact Formatting

User Contacts are stored in the engine in GAME Variables.

When a user is added to the contacts list, four GAME variables are created:

* **USER\_contact**, IE Aiko_contact. This is set to a value of 1.
* **USER\_group**, set to 1 by default. a value of 2 is set if this contact is a friend, 3 if they are a foe.
* **USER\_nick**, set to the username of the contact. This is a little redundant and should eventually be removed, but it is depended upon by a large amount of warp-related CAOS and in the interest of converting the code to be Albian Warp compatible as quickly as possible, it was left intact.

After the contact has been created, one more contact variable is set:

* **USER_status**, set to "online" or "offline" This value is actually set by the Albian Warp client, rather than a script.

With the exception of the \_group value, these variables should not be modified by the user or third party agents.

## NET: Command Replacement CAOS Functions

A large portion of the NET: CAOS Commands have been replaced in the Albian Warp code by CAOS Functions. These are portions of code that are loaded from scripts into GAME variable strings upon injection, and then executed with the command, `CAOS`. These can be a bit ugly and annoying to copy-paste, but they are vastly easier to read than copy-paste the entire blocks of code themselves. Attempts were made to keep the CAOS function behavior as close to the original NET: commands as possible, but some of them behave a little differently.

Use of the CAOS Functions in your own agents is highly encouraged, but avoid editing the CAOS function scripts or strings directly as several agents depend on them to function. If you have found a way to improve the NET: CAOS functions, please consider contributing directly to the Albian Warp project repository so everyone can benefit.

Here are the NET: Commands that have been replaced, along with the CAOS lines that replace them:

* **NET: RUSO**: `net: ruso va30` >> `sets va30 caos 0 0 0 0 game "f_aw_ruso" 0 0 va99`
	* Sets va30 to the name of a random user
* **NET: ULIN**: `setv va30 net: ulin "username"` >> `setv va30 stoi caos 0 0 "username" 0 game "f_aw_ulin" 0 1 va99`
	* Sets va30 to 0 if the user is offline, 1 if they are online, and 2 if there is no such user.
* **NET: WHON**: `net: whon va30` >> `sets va99 caos 0 0 va30 targ game "f_aw_whon" 0 0 va99`
	* Adds the user specified in va30 to the Who's Wanted Register for the target agent
* **NET: WHOF**: `net: whof va30` >> `sets va99 caos 0 0 va30 targ game "f_aw_whof" 0 0 va99`
	* Removes the user specified in va30 from the Who's Wanted Register for the target agent
* **NET: WHOZ**: `net: whoz` >> `sets va99 caos 0 0 targ 0 game "f_aw_whoz" 0 0 va99`
	* Clears all usernames from the Who's Wanted Register for the target agent
* **NET: WHOD**: `net: whod` >> `outs caos 0 0 0 0 game "f_aw_whod" 0 0 va99`
	* Dumps debug infomation about the Who's Wanted Register to the debug log
	* The original version of this command dumped the information to the output stream instead of the debug log

## Direct Messaging Agents (DMAs)

Most communication done through Albian Warp is made possible by Direct Messaging Agents. These are simply agents that are created with the classifier `1 1 35753` and are set with the NAME variable "aw_recipient" containing the username of the person that the message will be sent to. They contain no scripts, but they can contain any other NAME variables that the recieving scripts need to process the message.

When an outgoing DMA is created, the client picks it up, converts it to a JSON format, and sends it to the server. The server then sends it to the client of the person it is intended to recieve if they are online, or stores the message on the server until the next time that person comes online. Once the recieving client recieves the message, it turns it into an agent and injects it into the game. This agent is classified as `1 1 35754` containing the variables "aw\_sender" containing the username of the sender, and "aw\_date" containing the server date and time it was sent. This in the same format as is returned by the CAOS command `RTIM` and can be parsed with `RTIF`. It also carries over any NAME variables that were defined on that agent by the sender.

Once the DMA is injected into the recipient's game, an agent that is intended to handle that DMA will find it and use the information stored in the NAME variables to do something, such as display a message.

###Sending Mail

The Mail Message system within the game was recreated using DMAs. When a player composes and sends a mail message, an agent is generated by script 1 1 205 1101 (found in 'mailgrendel.cos') that looks something like this:

```
new: simp 1 1 35753 "blnk" 1 0 0
sets name "aw_recipient" "potato"
sets name "type" "mail_message"
sets name "subject" "A Subject Line"
sets name "message" "Here is a message. It is a very good message"
```

Assuming the sender (bob) and the recipient both have their game and client open, the user 'potato' will soon have an agent injected into their game by the client that looks something like this:

```
new: simp 1 1 35754 "blnk" 1 0 0
sets name "aw_sender" "bob"
sets name "aw_date" 999999
sets name "type" "mail_message"
sets name "subject" "A Subject Line"
sets name "message" "Here is a message. It is a very good message"
```

Although potato won't know the agent has been injected (unless they are staring at the output of their client window), the Message Interpreter Agent (1 1 206, found in 'mailgrendel.cos') runs its timer script every so often to enumerate over agents classed as 1 1 35754 and check if name variable "type" exists and is equal to "mail_message". Once it finds one that is, it uses it to create a Message Agent which shows up in potato's inbox for them to read and reply to if desired.

You can fake a messge to yourself by injecting the second block of code into your game yourself and altering the fields however you wish. Avoid injecting the first block unless necessary for testing purposes, as it is very easy to spam someone this way.

##Real-Time Direct Messaging Agents (RTDMAs)

RTDMAs are very similar to DMAs. They are set with the NAME variable "aw_recipient", and when recieved are stamped with the variables "aw\_sender" containing the username of the sender, and "aw\_date" containing the server date and time it was sent. However, RTDMAs are transmitted using a websocket connection, meaning there is much less delay in sending and recieving them. This makes them more suitable for applications such as chatting. However, unlike DMAs, RTDMAs are not stored on the server for later retrieval by offline users. If an RTDMA is sent to a user who is offline, it simply disappears.

RTDMAs use the classifiers `1 1 35755` for sending and `1 1 35756` for recieving.

### Chatting

The Chat system was rebuilt in the game using RTDMAs. It is one of the most complex systems in the game, consisting of several different message types.

All RTMDAs used for the Chat system require two additional NAME variables: `ChatID` and either `REQU` or `CHAT`.  `ChatID` is a string that is randomly generated when a player first sends out a chat invitation and is used to identify all messages and notifcations related to that chat session until all chatters have left the session. `CHAT` and `REQU` denote the type of message being sent so the script can direct the contents to the type of script.

Incoming `REQU` RTDMAs are intepreted by the timer script of the Chat Request Interpreter (Classified as `1 1 213`). Similarly, incoming `CHAT` RTDMAs are interpreted by the timer script of the Chat Message Interpreter (`1 1 209`). To conserve resources, this agent's tick varies based on whether or not a chat window is open. Both of these scripts can be found in the file "aw_chat_replacements.cos".

####REQU Messages

There are eight types of `REQU` RTDMAs:

* **Request**: Recieved when someone requests a chat with the user. This creates the Chat Request Indicator agent (1 1 214) that will appear on the user's screen with a 15 second timer. Depending on the user's response, either an Accept, Decline, or Timeout RTDMA is sent back to the requesting user.

* **Accept**: Recieved when someone accepts a chat request that the user has sent. This creates the Chat Window Agent (1 1 210) and the conversation can begin.

* **Decline**: Recieved when someone declines a chat request that the user has sent. This sets the appropriate message on the comms screen.

* **Timeout**: Recieved when someone ignores a chat request that the user has sent, allowing the 15 second timer to tick down to zero. This sets the appropriate message on the comms screen.

* **Join Existing Chat**: Recieved when someone invites the user to a chat session that is already in progress. Just like a normal Request RTDMA, this creates the Chat Request Indicator agent. However, these RTDMAs can contain several pairs of additional NAME variables named "chatter1\_UserID" and "chatter1\_Nickname" (or chatter2\_UserID", etc) containing the usernames of each chatter that is already in the chat. If the user accepts the invitation, an Invite Accept RTDMA is send back, the Chat Window is created and is populated with the names of the other users in the chat. If the user declines or ignore the invitations, an Invite Decline or Invite Timeout RTDMA is sent back accordingly.

* **Invite Accept**: Recieved when someone the user has invited to an existing chat accepts their invitation. This will add the new chatter to the user's chatter list and then send each other user in the chatroom a `CHAT`: Update RTDMA to ensure that the new chatter is also added to thier lists.

* **Invite Decline**: Recieved when someone the user has invited to an existing chat has declined the invitation. Prints an appropriate message in the chat window to alert the user.

* **Invite Timeout**: Recieved when someone the user has invited to an existing chat has ignores a chat request that the user has sent, allowing the 15 second timer to tick down to zero. Prints an appropriate message in the chat window to alert the user.

####CHAT Messages

There are three types of `CHAT` RTDMAs:

* **Message**: Recieved when someone the user is in a chat session with sends a message. Contains the text of the message in the NAME variable "Chat Message". Displays the message in the chat window.

* **Chatter go Bye Bye**: Recieved when someone the user is in a chat session with closes the chat window. Removes the user from the chatter list and displays a message that they have left the chat.

* **Update**: Recieved when a new chatter (that the user did not personally invite) joins a chat session that the user is in. Adds the user to the chatter list.
