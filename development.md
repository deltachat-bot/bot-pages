---
title: Developer information
---

# Developer information

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

🚧 TODO: Describe the event-based nature of the core engine, name important core events. Exemplify one simple event-answer-roundtrip.

