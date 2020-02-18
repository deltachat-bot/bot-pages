---
title: How to Run a Delta Chat Bot
---

# How to Run a Delta Chat Bot

To run a bot, you don't need to be a programmer. 
You don't even need to own or rent a server.
A bot runs on any computer.

Any bot has a documentation about how to setup and use it. 
For our example bots, it is in the README.md in their respective [GitHub repository](https://github.com/deltachat-bot/).

## Internet Connection and Dependencies

For running any Delta Chat bot, there is one basic requirement: That a network connection to your email provider is possible via IMAP and SMTP.

Next, you need an installation of the runtime (programming language) that the bot uses (e.g. [Python](https://www.python.org/downloads/), [rustup](https://www.rust-lang.org/tools/install), or [NodeJS](https://tecadmin.net/install-nodejs-with-nvm/)), if you don't have that already.
Check out the documentation of the bot of your choice, you should find the information there.

Finally you need to install the bot itself, and find a way to keep it running.
One way is to restart it automatically in case it stops - a useful tool for this is e.g. [forever for NodeJS applications](https://www.npmjs.com/package/forever).
You can do something similar with [Python](https://www.alexkras.com/how-to-restart-python-script-after-exception-and-run-it-forever/).
Another tool which may help you is [tmux](https://github.com/tmux/tmux/wiki), which can keep a shell open, even if you close the SSH session.
Whether you need these tools depends highly on your setup.

## Every Bot Needs an Email Address

The bot's documentation will tell you if there is anything else to prepare.
For example, a Delta Chat bot always uses one or more email addresses.
The documentation specifies how to tell the bot about its email address and password.
After all, a Delta Chat bot is basically a special purpose email client.

Tip: choose an email provider that doesn't (or does only very liberally) filter, block or delay incoming or outgoing emails. With good email providers, messages reach the chat partners within a few seconds.

🚧 **TODO**: Links with short descriptions to example bots once they're ready.

