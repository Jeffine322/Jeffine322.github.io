Scholar's Notes v29 - 需求锁定版

这版按你重新对齐的需求做了固定：

1. 整体框架不动，保持参考图的手账风格。
2. 最近文章区域永远保留 3 个卡片：
   - 免杀卡片
   - 渗透卡片
   - 开源卡片
3. 每个卡片只显示对应分类最新一篇笔记：
   - tag: 免杀
   - tag: 渗透
   - tag: 开源
4. 如果某个分类没有笔记：
   - 卡片不删除
   - 猫图不删除
   - 只把文字内容留空
5. 右侧“近期更新”按时间倒序显示最近 5 篇上传笔记名。
6. 四个主题入口和右侧 Tags 都跳到 category.html 分类页。
7. 需要上传三个文件到仓库根目录：
   - index.html
   - post.html
   - category.html

文章放置方式：
在仓库根目录新建 posts 文件夹，上传 Markdown 笔记，例如：

posts/2026-05-31-web-src-api.md

文件内容示例：

---
title: Web / SRC 信息搜集到 API 分析学习流程
date: 2026-05-31
tag: 渗透
summary: 适用场景：授权测试、靶场环境、自有资产梳理。
cover: ./assets/cat-cry.png
---

# Web / SRC 信息搜集到 API 分析学习流程

这里写正文。

注意：
- tag 必须写 免杀 / 渗透 / 开源 / 其他 之一。
- date 用 YYYY-MM-DD。
- cover 可以不写，不写会按分类自动选猫图。
