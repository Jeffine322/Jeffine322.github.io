---
title: 学习使用 Hugging Face
date: 2026-02-05
tag: 其他
summary: 用 Hugging Face 白嫖小模型，在云 IDE / GitHub Actions 上跑摘要、文本分类和舆情监控，缓解长文本超上下文的问题。
cover: ./assets/cat-note.png
---

# 学习使用 Hugging Face

# 前言

今天好累写写代码养生一天。今天重写rss日报的时候在想怎么解决爬取的信息内容超过大模型上下文的问题，无意间发现了hugging face原来这么好用。

在hugging face上找了摘要生成的模型，效果显著，资源也消耗不多。



# 云ide

![image-20260204212707739](./assets/posts/学习使用HuggingFace/image-20260204212707739.png)

在腾讯云ide上运行模型，这里运行的是摘要任务



![image-20260204213030596](./assets/posts/学习使用HuggingFace/image-20260204213030596.png)

只需要按照torch即可



![image-20260204213730786](./assets/posts/学习使用HuggingFace/image-20260204213730786.png)

等待模型下载



![image-20260204214141459](./assets/posts/学习使用HuggingFace/image-20260204214141459.png)

然后就成功的生成了摘要，着和docker有什么区别，非常方便啊



![image-20260204214506771](./assets/posts/学习使用HuggingFace/image-20260204214506771.png)

最后用了类似的方式对rss的description进行摘要一定程度上缓解了超出上下文长度的问题。



![image-20260204224532630](./assets/posts/学习使用HuggingFace/image-20260204224532630.png)

类似的模型在这里可以选择



![image-20260204224555025](./assets/posts/学习使用HuggingFace/image-20260204224555025.png)

随便点开一个就会有使用代码



# 魔搭社区

国内无法直接访问hugging face，可以访问魔搭但是魔搭的依赖管理很复杂，用魔搭老是报错缺东西



![image-20260204214428547](./assets/posts/学习使用HuggingFace/image-20260204214428547.png)

例如qwen 0.6B的，其中给了代码



![image-20260204215357255](./assets/posts/学习使用HuggingFace/image-20260204215357255.png)

但是就是会各种报错



# huggingface

![image-20260204225006647](./assets/posts/学习使用HuggingFace/image-20260204225006647.png)

改为hugging face的代码



![image-20260204230423188](./assets/posts/学习使用HuggingFace/image-20260204230423188.png)

然后部署到action上，就生成结果了，模型参数很小只有0.6B



## 文本总结

![image-20260204231455136](./assets/posts/学习使用HuggingFace/image-20260204231455136.png)

如上面这样



![image-20260204231639418](./assets/posts/学习使用HuggingFace/image-20260204231639418.png)

算上依赖下载，花费2分钟完成



## 文本分类

![image-20260204231753233](./assets/posts/学习使用HuggingFace/image-20260204231753233.png)

例如量化交易中的文本分类，根据新闻将btc市场分类为利好或者利空



![image-20260204232152011](./assets/posts/学习使用HuggingFace/image-20260204232152011.png)

到这里其实能利用action写一个简单的基于新闻的量化交易策略了。



## 舆情监控

![image-20260205000007880](./assets/posts/学习使用HuggingFace/image-20260205000007880.png)

请求某一个论坛的新闻页面



![image-20260205000039816](./assets/posts/学习使用HuggingFace/image-20260205000039816.png)

传入提示词中让其判断是否有涉及国内



![image-20260204235856458](./assets/posts/学习使用HuggingFace/image-20260204235856458.png)

这个生成花了20多分钟，毕竟是免费。



## 白嫖的极限

![image-20260204234407853](./assets/posts/学习使用HuggingFace/image-20260204234407853.png)

根据测试3B的模型单次提问就要等4分钟左右了，大概就是极限了。



# 总结

给我一种大模型的docker的感觉，比ollama的使用场景更多。



![image-20260205000243512](./assets/posts/学习使用HuggingFace/image-20260205000243512.png)

包含了各种各样奇怪的模型。
