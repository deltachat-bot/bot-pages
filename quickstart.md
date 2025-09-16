---
title: Write a Bot
---

# Write a Bot

For a start, let's build an "echo" bot in python. For every message you send to it, it will reply with that same text.
You can look at this [example repository](https://github.com/deltachat-bot/echo) for solutions in different programming languages!
{: .box }

## Setup virtual environment

First let's setup a Python virtual environment where we can safely
install and test the dependencies we need:

```sh
pip install virtualenv
virtualenv .venv
source .venv/bin/activate
```

## Installing dependencies

Now that we have a virtual environment active for testing, let's
install the dependencies we need to create our bot:

```sh
pip install deltabot-cli
```

**NOTE:** `deltabot-cli` is a high-level bot framework library
that simplifies writing bots and speeds the development process.

## Creating a Python script file

Now create a file called `echobot.py` where you will put the code
of your bot. The first thing we will put inside is the import of
the dependencies we will be using:

```python
from deltabot_cli import BotCli
from deltachat2 import MsgData, events
```

## Setting up the bot CLI

Now let's setup the bot's command line interface:

```python
cli = BotCli("echobot")
```

This creates a `cli` object that will use `echobot` as default
folder name for the bot configuration folder
(ex. `~/.config/echobot/` on Linux).

## Handling incoming messages

With the `cli` object we can now register event handlers:

```python
@cli.on(events.NewMessage)
def echo(bot, accid, event):
    msg = event.msg
    reply = MsgData(text=msg.text)
    bot.rpc.send_msg(accid, msg.chat_id, reply)
```

The line `@cli.on(events.NewMessage)` means the following function
will be called when a new message is received. The function
receives the bot instance, the account ID where the message was
received and the event object offering `event.command` and
`event.payload`, strings with the command issued by the user and
the command's payload, for example if the user sent a message with
text: `/uppercase hello`, then `event.command == "/uppercase"` and
`event.payload == "hello"`. For our echo bot we don't need any
commands and we can access the incoming message object directly
via `event.msg`.

With `reply = MsgData(text=msg.text)` we create a reply object
with the same text as the incoming message.

We use `bot.rpc.send_msg(accid, msg.chat_id, reply)` to send the
reply message to the same chat where the incoming message was
received. The `bot.rpc` object allows you to interact with the
[chatmail core API](https://github.com/chatmail/core/blob/main/deltachat-jsonrpc/src/api.rs).

## Starting the CLI

When our `echobot.py` script is run we need to start the bot's CLI
so events are processed:

```python
if __name__ == "__main__":
    cli.start()
```

This powers our bot with a fully featured CLI with options to
configure and run our bot!

## Full source code

That is it! We have a fully functional echo-bot, here is the
complete source code of our `echobot.py` script:

```python
from deltabot_cli import BotCli
from deltachat2 import MsgData, events

cli = BotCli("echobot")

@cli.on(events.NewMessage)
def echo(bot, accid, event):
    msg = event.msg
    reply = MsgData(text=msg.text)
    bot.rpc.send_msg(accid, msg.chat_id, reply)

if __name__ == "__main__":
    cli.start()
```

## Testing the bot

Now it is time to test our new bot program!

### Configure

First let's create the bot's chatmail profile:

```sh
python ./echobot.py init DCACCOUNT:https://nine.testrun.org/new
```

**TIP:** We will be using nine.testrun.org chatmail relay,
you can register a new account in many other existing
[chatmail relays](https://chatmail.at/relays)

After the profile is created we can tweak the bot's display name,
avatar and the status/description message:

```sh
python ./echobot.py config displayname "My Echo Bot"
python ./echobot.py config selfstatus "Hi, I am an echo-bot"
python ./echobot.py config selfavatar "./bot-avatar.png"
```

### Get the bot's chat invitation link

To get in contact with the bot we need to get the bot's invite link:

```sh
python ./echobot.py link
```

The bot's invite link will be printed in the shell, open it with
your Delta Chat client or in the browser and scan the QR code.
The chat invitation will be accepted by the bot once you let the
bot process incoming messages, keep reading to the next step.

### Run the bot

Finally the time has come to start chatting with your bot!
We instruct the bot to go online and process messages using
the command:

```sh
python ./echobot.py serve
```

Now go to your Delta Chat, you should already have a chat with
the bot, try sending "hi" or any other text message to it!

### Next steps

Congratulations! you got the basics of creating and running a
Delta Chat bot!

Further readings:

* [deltabot-cli page](https://github.com/deltachat-bot/deltabot-cli-py)
* [deltachat2 library used by deltabot-cli](https://github.com/adbenitez/deltachat2)
* [echo-bot implementation examples in different languages](https://github.com/deltachat-bot/echo)

**NOTE:** `deltabot-cli` and `deltachat2` libraries are in
development and proper documentation is missing, for now you will
have to read their provided examples and their source code.
For a list of the available JSON-RPC methods [click here](https://github.com/chatmail/core/blob/main/deltachat-jsonrpc/src/api.rs)
