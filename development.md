---
title: Developer Information
---

# Developer Information

Building your own Delta Chat bot is easy.
Read on for some basic information and maybe have a look at our [example bots](https://deltachat-bot.github.io/public-bots/).
{: .box }

## API

Delta Chat bots use [chatmail core](https://github.com/chatmail/core), the engine doing most of the work.

Your job as bot author is to use the output of that engine, and to give it instructions. You won't parse any email message, check connections, or care for encryption, that's all the engines job.

The core engine can be used natively from Rust, through a [JSON-RPC API](https://jsonrpc.delta.chat/) from many other languages. There is also a [C-API](https://c.delta.chat) that can be used from C and C++. (Please note that the Rust and JSON-RPC API are still subject to change, while the C-API is considered stable.)

If you need a starting point for your bot look at the [echo bot](https://github.com/deltachat-bot/echo), which has examples for getting started in multiple languages.

(If you language of choice misses here, please consider to write bindings to the API for it!)


## Background

The Delta Chat core engine manages all network connections and watches for activity.
It also provides ways to interact with known data (e.g. reading messages) as well as to create new data (e.g. managing contacts, sending messages).

Starting the Delta Chat core engine means to start threads that connect to your email server and work on jobs like watching for new messages, fetching messages, and sending messages.
Any activity that the core engine considers worthwhile knowing is broadcasted through events.
In order to stay on top of what's happening, your bot must listen for events it wants to handle.

#### Events

Here's a [list of all available events](https://c.delta.chat/group__DC__EVENT.html) with descriptions of their meaning.

Most events carry a payload of one or two arguments, which contain the associated data. That might be an incoming message, but it might also be a number that signifies the progress of an operation.

One useful example for an event and its payload would be [`DC_EVENT_INCOMING_MSG`](https://c.delta.chat/group__DC__EVENT.html#ga3f0831ca83189879a2f224b424d8b58f), the event emitted when a new message for an existing chat has arrived.
Its payload are the ID of the chat for which the message arrived, and the ID of the message itself.
If a payload argument is 0, an error happened. Errors are broadcasted through different events.
To get hands on the actual chat and message, your code then will have to fetch it from the database by calling the API.

**Note**: the high-level bot frameworks handle the `DC_EVENT_INCOMING_MSG` for you and provide you the incoming message data,
this is only mentioned to explain the background.
{: .notification }