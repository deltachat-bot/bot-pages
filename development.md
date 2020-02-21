---
title: Developer Information
---

# Developer Information

Building your own Delta Chat bot is easy.
Read on for some basic information and maybe have a look at our [example bots](howto.html#bots).

### API

Delta Chat bots use [deltachat-core-rust](https://github.com/deltachat/deltachat-core-rust), the engine doing most of the work.

Your job as bot author is to use the output of that engine, and to give it instructions. You won't parse any email message, check connections, or care for encryption, that's all the engines job.

The core engine can be used natively from rust, and through a [C-API](https://c.delta.chat) from C and C++. (Please note that the rust API is still subject to change, while the C-API is considered stable.)

Additionally there are bindings to the C-API for the following languages:
* Python: [https://py.delta.chat](https://py.delta.chat)
* NodeJS: [https://github.com/deltachat/deltachat-node](https://github.com/deltachat/deltachat-node)
* Golang: [https://github.com/hugot/go-deltachat](https://github.com/hugot/go-deltachat)

If you language of choice misses here, please consider to write bindings to the API for it! Using FFI it shouldn't be too hard.


### Background

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

See [the code of this python example bot](https://github.com/deltachat-bot/deltabot/blob/master/src/deltabot/cmdline.py#L110) for context code and how to maybe listen for both events at once.


## Examples

### NodeJS echo bot

As a very easy, high level example here's an implementation of a simple echo bot in NodeJS.
It uses a small library called [`deltachat-node-bot-base`](https://github.com/deltachat-bot/deltachat-node-bot-base) that abstracts the bindings for NodeJS a little more and makes it even easier to build a bot.

Install the dependency using NPM in a fresh directory:

```bash
npm install git://github.com/deltachat-bot/deltachat-node-bot-base
```

Configure the bot by writing its email-address and password into `config/local.json` like this:

```json
{
  "email_address": "bot@example.net",
  "email_password": "secretandsecure"
}
```

Now put the following code into `./echobot.js` and run it with `node echobot`

```javascript
// Require deltachat from the bot-base library.
// Also require the log() function as a simple helper to have our logged lines
// appear the same as the logged lines from the library.
const { deltachat, log } = require('deltachat-node-bot-base')

// Start the deltachat core engine and handle incoming messages.
deltachat.start((chat, message) => {
  // Get the message from the API.
  const messageText = message.getText()
  log(`Received a message for chat ${chat.getName()}: ${messageText}`)

  // Look at the number of contacts in the chat of this message. If we get
  // exactly one contact, it is a 1-on-1 (aka "single") chat. (Technically, it
  // might also be a group chat with only ourselves as member, but then we
  // couldn't have received this message.)
  if (deltachat.getChatContacts(chat.getid()).size === 1) {
    // We reply by repeating the same text with a prefix.
    deltachat.sendMessage(chat.getId(), `You said: ${messageText}`)
  } else {
    // There are not chats with zero members. Thus, this appears to be a group chat.
    // We reply only if the message starts with "Bot".
    if (messageText.match(/^bot[:, ]+/i)) {
      const contact = deltachat.getContact(message.getFromId())
      const displayName = contact.getDisplayName()
      deltachat.sendMessage(chat.getId(), `${displayName} said: ${messageText}`)
    }
  }
})
```

### NodeJS web publishing bot

Have a look at [public-groups-bot](https://github.com/deltachat-bot/public-groups-bot) and its underpinning library for web-interacting bots, [deltachat-node-webbot-base](https://github.com/deltachat-bot/deltachat-node-webbot-base).

🚧 Describe these bots and their functionality more verbosely.
