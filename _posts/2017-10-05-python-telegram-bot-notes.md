---
layout: post
title: Python Telegram Bot Notes
---

## 需求
鉴于RSS式微，身边一些新闻站点不再提供此功能，[fivefilters.org](http://fivefilters.org/)可实现任意网页RSS输出及全文输出，数年使用，效果满意。但有些网站仅限内网访问，故需要在内网运行类似的服务。此外，fivefilters.org的任意网页RSS输出只支持简单的is/class过滤，不支持wildcard，对一些代码风格凌乱的网站束手无策。

当然，运行在内网的RSS源抓取工具无法提供RSS阅读器可用的RSS链接。我们的目标是追踪内网网站的新闻更新，需要解决两个问题：
1. 监测网页，获取更新(该操作只能在内网完成);
2. 将更新推送给用户(自己，将来可以扩展到公网上的其他用户)。

## 方案
自己常用的服务中，满足全平台消息推送并提供良好扩展接口的好像只有[Telegram](https://telegram.org/)。简单看了一下Telegram的Bot API，主要使用Web接口，本着DRY(don't repeat yourself)(其实主要是懒)的原则，还是找个封装好的接口吧。
几种常用的编程语言，自己能看懂、修改的只有Python和PHP，JavaScript，Java，Ruby，.NET/C#只能呵呵。开源的Telegram Bot封装里面，[Python Telegram Bot](https://github.com/python-telegram-bot/python-telegram-bot)在GitHub上star最多，就它了。

之前写空气质量Bot的时候用过这个库，感觉嘛，Wiki质量不行，从最简单的入手到成品之间没有可参考的代码讲解，学习曲线太陡(浪费了一个下午，我会乱讲？)。

## 笔记
代码一定要加Logging，不然调试的时候想死的心都有，而且特别浪费时间。

BeautifulSoup4 find的结果type是class，bot.send_message(chat_id=[你的Telegram ID], text=str([BeautifulSoup4 find的结果]), parse_mode=telegram.ParseMode.HTML)。

## 参考
1. [https://python-telegram-bot.org/](https://python-telegram-bot.org/)
2. [https://github.com/python-telegram-bot/python-telegram-bot/wiki/Extensions-%E2%80%93-JobQueue](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Extensions-%E2%80%93-JobQueue)


    