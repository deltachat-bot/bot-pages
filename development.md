---
title: Developer Information
---

# Developer Information

Building your own Delta Chat bot is easy.
Read on for some basic information and maybe have a look at our [example bots]({{ site.baseurl }}howto.html#bots).
{: .box }

## API

Delta Chat bots use [deltachat-core-rust](https://github.com/deltachat/deltachat-core-rust), the engine doing most of the work.

Your job as bot author is to use the output of that engine, and to give it instructions. You won't parse any email message, check connections, or care for encryption, that's all the engines job.

The core engine can be used natively from rust, and through a [C-API](https://c.delta.chat) from C and C++. (Please note that the rust API is still subject to change, while the C-API is considered stable.)

For the following languages there are bindings to the C-API:
* Python: [https://py.delta.chat](https://py.delta.chat)
* NodeJS: [https://github.com/deltachat/deltachat-node](https://github.com/deltachat/deltachat-node)
* Golang: [https://github.com/hugot/go-deltachat](https://github.com/hugot/go-deltachat)

If you need a starting point for your bot look at the [echo bot](https://github.com/deltachat-bot/echo), which has examples for getting started in multiple languages. (C, go, nodejs, python, rust)

(If you language of choice misses here, please consider to write bindings to the API for it! Using FFI it shouldn't be too hard.)


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

In Python this might look like this (with a configured account and running threads):
```python
while 1:
    event = account._evlogger.get_matching("DC_EVENT_INCOMING_MSG")
    if event[2] != 0:
        msg = account.get_message_by_id(event[2])
        maybe_do_something(msg)
```

Messages that don't belong to an existing chat must be filtered from a different event: [`DC_EVENT_MSGS_CHANGED`](https://c.delta.chat/group__DC__EVENT.html#ga0f52cdaad70dd24f7540abda6193cc2d). The code must check that the message is put into the chat called "deaddrop", which is the bucket for all content messages that don't belong to an existing chat yet (e.g. if someone sends you an initial message).

In Python:

```python
while 1:
    event = account._evlogger.get_matching("DC_EVENT_MSGS_CHANGED")
    if event[2] != 0:
        msg = account.get_message_by_id(event[2])
        if msg.chat().is_deaddrop():
            do_something(msg)
```

**Note**: the Python bindings for Delta Chat provide convenient abstractions over these events. You don't have to write code like this for a real world bot, this is only shown to explain the background.
{: .notification }


Real world bots are [showcased in the Delta Chat forum](https://support.delta.chat/c/bots). Go ahead and look at their source code, too!
{: .notification }
