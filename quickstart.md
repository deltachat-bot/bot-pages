---
title: Quick Start
---

# Quick Start

For a start, let's build an "echo" bot in python. For every message you send to it, it will reply with that same text.
You can look at this [example repository](https://github.com/deltachat-bot/echo) for solutions in different programming languages!
{: .box }

To install the `deltachat` python package, see the [deltachat python
documentation](https://py.delta.chat/install.html). You can import it like
this:

```python
import deltachat
```

## Setting up an Account Object

A Delta Chat usually starts with an account object, the account your bot will
use to communicate with others. It is initialized with a path to the database
where it will store its data. Make sure that path exists:

```python
import deltachat

ac = deltachat.account.Account("/tmp/bot_db/")
```

Now, to login to the email account, the bot needs its own email address + a
password. With most servers, this will be enough to get started (some servers
might require [additional configuration](https://providers.delta.chat)). Let's
get username + password from environment variables, configure, and run the
account:

```python
import deltachat
import os

os.getenv("DELTACHAT_DB_PATH", default="/tmp/bot_db/")
ac = deltachat.account.Account(db_path)

addr = os.getenv("DELTACHAT_BOT_ADDR")
password = os.getenv("DELTACHAT_BOT_PASSWORD")

ac.run_account(addr=addr, password=password)
ac.wait_shutdown()
```

[`account.run_account()`](https://py.delta.chat/api.html#deltachat.account.Account.run_account)
will run the account, and configure it if necessary (e.g. look up the mail
servers of your provider).

[`account.wait_shutdown()`](https://py.delta.chat/api.html#deltachat.account.Account.wait_shutdown)
waits until the account quits running for some reason. This is necessary here
so the script doesn't quit directly, but stays open as long as the bot is
running. We want the bot to reply to our messages forever, after all.

## Receiving Messages and Other Events

The account object has an event thread running; if an event happens, it will
call a hook. You can override this hook in an account plugin with your own
function to let a bot react to something, e.g. a message.

An account plugin is just a class, which we add to the account object:

```
class EchoPlugin:
    pass

ac.add_account_plugin(EchoPlugin)
```

Now this in itself is not very useful yet. But we can override
`ac_incoming_message` in the plugin, so we can hook into incoming messages:

```
class EchoPlugin:
    @deltachat.account_hookimpl
    def ac_incoming_message(self, message):
        print(message.text)
```




## Sending a Message














## API

Delta Chat bots use [deltachat-core-rust](https://github.com/deltachat/deltachat-core-rust), the engine doing most of the work.

Your job as bot author is to use the output of that engine, and to give it instructions. You won't parse any email message, check connections, or care for encryption, that's all the engines job.

The core engine can be used natively from rust, and through a [C-API](https://c.delta.chat) from C and C++. (Please note that the rust API is still subject to change, while the C-API is considered stable.)

For the following languages there are bindings to the C-API:
* Python: [https://py.delta.chat](https://py.delta.chat)
* NodeJS: [https://github.com/deltachat/deltachat-node](https://github.com/deltachat/deltachat-node)
* Golang: [https://github.com/hugot/go-deltachat](https://github.com/hugot/go-deltachat)

Additionally there are two small libraries for NodeJS that provide a higher level API to write bots:
* [deltachat-node-bot-base](https://github.com/deltachat-bot/deltachat-node-bot-base) — for writing bots (duh!),
* [deltachat-node-webbot-base](https://github.com/deltachat-bot/deltachat-node-webbot-base) — for web-interacting bots (bases on the former).

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

## Code example

To visualize how simple writing a Delta Chat bot can be, here's a stripped down implementation of an echo bot in NodeJS:

```javascript
const { deltachat } = require('deltachat-node-bot-base')

deltachat.start((chat, message) => {
  const messageText = message.getText()
  if (messageText && chat.isSingle()) {
    deltachat.sendMessage(chat.getId(), `You said: ${messageText}`)
  }
})
```

This code is extracted from the README of the aforementioned library [`deltachat-node-bot-base`](https://github.com/deltachat-bot/deltachat-node-bot-base). For more details and usage instructions see there.

Real world bots are [showcased in the Delta Chat forum](https://support.delta.chat/c/bots). Go ahead and look at their source code, too!
{: .notification }
