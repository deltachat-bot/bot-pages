---
title: How to run a Delta Chat Bot
---

# How to Run a Delta Chat Bot

To run a Delta Chat bot you don't need to be a programmer or experienced sysadmin.
Basically any computer will do, as long as it can connect to your email provider.
For testing purposes that may well be your local machine. :)


### Requirements
The details depend on the bot of your choice.
Please read its documentation (see below for links).

In general you need:
* a connection to the internet,
* an installation of the programming language that the bot uses (e.g. Python or NodeJS),
* and some additional software packages (don't be afraid, the installation won't be complicated).

Some bots will have those dependencies packaged into a container image, then you need only the possibility to run a container (e.g. with docker or podman).

### Configuration
Each Delta Chat bot requires at least one email address and a password — after all a Delta Chat bot is basically just an special purpose email client.

There might be additional preparations steps described in the bot's documentation, please don't forget to read it.

Choose an email provider that doesn't filter, block or delay incoming or outgoing emails. With good email providers, messages reach the chat partners within a few seconds, bad providers can cause long delays.
{: .tip }

### Running
If you don't use a container image you'll start the bot by executing a command. In order to keep it running when you log out, maybe use a tool like [tmux](https://github.com/tmux/tmux/).

## Bots

🚧 **TODO**: Links with short descriptions to example bots once they're ready.

