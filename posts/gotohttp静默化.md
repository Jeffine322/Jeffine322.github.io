---
title: 使用 AI 把 gotohttp 静默化：去 GUI 去 UAC
date: 2026-05-13
tag: 免杀
summary: 用 AI 的 IDA / x64dbg MCP 自动分析并 patch gotohttp，去掉 GUI 托盘和 UAC，做成静默版并测免杀。
cover: ./assets/cat-angry.png
---

# 使用 AI 把 gotohttp 静默化：去 GUI 去 UAC

# 前言

之前古法patch把rustdesk的gui全删了，弄成静默的了。我在想gotohttp能不能，我之前古法调试分析了很久没成功后面放弃了。

现在有ai直接上ai



# mcp

![image-20260513210152728](./assets/posts/gotohttp静默化/image-20260513210152728.png)

idapro的mcp



![image-20260513210222333](./assets/posts/gotohttp静默化/image-20260513210222333.png)

x64dbg的mcp



# patch

![image-20260513210253907](./assets/posts/gotohttp静默化/image-20260513210253907.png)

它自动分析原因，自动patch。最终花费91分钟作用，成功把gotohttp所有的gui 托盘都去掉了，完全变成静默版本



![image-20260513210609052](./assets/posts/gotohttp静默化/image-20260513210609052.png)

然后再把uac给删了



# 效果

![image-20260513210651421](./assets/posts/gotohttp静默化/image-20260513210651421.png)

运行后生成ini文件



![image-20260513210740748](./assets/posts/gotohttp静默化/image-20260513210740748.png)

连接没有问题，全程静默



# 免杀效果

![image-20260513210956110](./assets/posts/gotohttp静默化/image-20260513210956110.png)

火绒没反应，360报蓝不删但会提醒，windows defender没反应。



# 总结

ai对古法lolbins思路的进一步自动化利用。只要思路够多，对agent设计prompt有足够理解，能做到很多奇奇怪怪的事情
