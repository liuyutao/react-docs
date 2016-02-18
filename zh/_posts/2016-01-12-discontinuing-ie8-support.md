---
title: "React DOM 不再支持IE8"
author: spicyj
---

从2013年发布以来， React已经支持了所有的流行浏览器，包括IE8及其以上版本。我们处理了很多老浏览器版本中的quirks模式，使其标准化，包括事件系统差异，这样你们的APP代码才不需要担心大多数的浏览器BUG.

今天，微软 [停止支持IE旧版本](https://www.microsoft.com/en-us/WindowsForBusiness/End-of-IE-support).从React v15开始，我们将停止React DOM 对IE8的支持。我们已经了解到，大部分的React DOM app已经不再支持旧版本的IE了，所以此变化应该影响不到大部分人。这个变化将帮助我们开发更快速，使React DOM 变得更好。（我们暂时不会主动移除IE8相关的代码，但我们不要重视新发现的此类相关Bug.如果您需要支持IE8,我们建议您保持React v0.14版本上。）

React DOM 将在可预见的将来继续支持IE9 及其以上版本。

