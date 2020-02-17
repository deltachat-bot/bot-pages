---
title: Developer information
---

# Developer information

Building your own Delta Chat bot is easy.
Read on for some basic information and maybe have a look at our example bots.
You may also copy and adapt them, if that suits you!

They're mostly licensed under the GPL, which allows you to use the code, and also to change it as long as you re-publish your result (at least on request).

## APIs you can use

Delta Chat bots use deltachat-core-rust, the engine doing most of the work. Your job as bot author is to use the output of that engine, and to give it instructions. You won't parse any email message, check connections, or care for encryption, that's all the engines job.

The core engine can be used natively, writing rust code. It also provides a [C-API](https://c.delta.chat) to enable its use from other languages. Besides C and C++ you currently can use the following languages, for which bindings to the core's API exist:
* Python: [https://py.delta.chat](https://py.delta.chat) (official)
* NodeJS: [https://github.com/deltachat/deltachat-node](https://github.com/deltachat/deltachat-node) (official)
* Golang: [https://github.com/hugot/go-deltachat](https://github.com/hugot/go-deltachat) (contributed)

If you language of choice misses here, please consider to write bindings to the API for it! Using FFI it shouldn't be too hard.

🚧 TODO: Describe the event-based nature of the core engine, name important core events. Exemplify one simple event-answer-roundtrip.

