---
title: Write a Bot
---

# Write a Bot

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
```

[`account.run_account()`](https://py.delta.chat/api.html#deltachat.account.Account.run_account)
will run the account, and configure it if necessary (e.g. look up the mail
servers of your provider).

## Receiving Messages

Now we can listen for incoming messages, and directly reply to them!

```python
ac.run_account(addr=addr, password=password)

while True:
    fresh = ac.get_fresh_messages()
    for message in fresh:
        print(message.text)
    ac.mark_seen_messages(fresh)
```

**Tip:** You can look at the [python
docs](https://py.delta.chat/api.html#deltachat.message.Message) to see what
else you can do with a deltachat.Message object.
{: .notification }

If we use `ac.get_fresh_messages()` to fetch new messages, we need to mark the
messages as seen afterwards, so we don't re-fetch it every time.

This generally works, but it is a bit primitive. You can also use event hooks:

### Optional: Use Hooks to Catch Events

New messages are just one of the different events which the core emits. Often
we want to listen for other events. We can use hooks for this.

The account object has an event thread running; if an event happens, it will
call a hook. You can override this hook in an account plugin with your own
function to let a bot react to something, e.g. a message.

An account plugin is just a class, which we add to the account object:

```python
class EchoPlugin:
    pass

ac.add_account_plugin(EchoPlugin)
```

Now this in itself is not very useful yet. But we can override the
`ac_incoming_message` method, which gets called on every incoming message
(duh). Then we can hook into incoming messages in the plugin:

```python
class EchoPlugin:
    @deltachat.account_hookimpl
    def ac_incoming_message(self, message):
        print("ac_incoming_message", message.text)
```

Here we don't need to mark the messages as notified, `ac_incoming_message` does
that for us.

**Tip:** You can find a full list of available hooks in
[hookspec.py](https://github.com/deltachat/deltachat-core-rust/blob/master/python/src/deltachat/hookspec.py)
in the python bindings.
{: .notification }

But now, without a while loop, the bot just quits after `run_account()` - how
can we keep it running?

[`account.wait_shutdown()`](https://py.delta.chat/api.html#deltachat.account.Account.wait_shutdown)
waits until the account quits running for some reason. This is necessary here
so the script stays open as long as the bot is running, instead of quiting
directly. We want the bot to reply to our messages forever, after all. So in
total:

```python
class EchoPlugin:
    @deltachat.account_hookimpl
    def ac_incoming_message(self, message):
        print("ac_incoming_message", message.text)

ac.add_account_plugin(EchoPlugin)
ac.run_account(addr=addr, password=password)
ac.wait_shutdown()
```

## Sending a Message

Now we can react to the incoming message, by replying with an exact echo of the
message. If you used the while loop, it looks like this:

```python
import deltachat
import os

os.getenv("DELTACHAT_DB_PATH", default="/tmp/bot_db/")
ac = deltachat.account.Account(db_path)

addr = os.getenv("DELTACHAT_BOT_ADDR")
password = os.getenv("DELTACHAT_BOT_PASSWORD")

ac.run_account(addr=addr, password=password)

while True:
    fresh = ac.get_fresh_messages()
    for message in fresh:
        if not message.is_system_message() and not message.chat.is_group():
            text = message.text
            message.chat.send_text(text)
    ac.mark_seen_messages(fresh)
```

Or if you want to use a hook function:

```python
import deltachat
import os

class EchoPlugin:
    @deltachat.account_hookimpl
    def ac_incoming_message(self, message):
        if not message.is_system_message() and not message.chat.is_group():
            text = message.text
            message.chat.send_text(text)


os.getenv("DELTACHAT_DB_PATH", default="/tmp/bot_db/")
ac = deltachat.account.Account(db_path)

addr = os.getenv("DELTACHAT_BOT_ADDR")
password = os.getenv("DELTACHAT_BOT_PASSWORD")

ac.add_account_plugin(EchoPlugin)
ac.run_account(addr=addr, password=password)
ac.wait_shutdown()
```

Voilà! Now you know the most important concepts of the deltachat core and can
start developing your own bots.

