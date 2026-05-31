Scholar's Notes v30

本版主要改 post.html 的文章渲染效果：

1. 代码块按文章 tag 自动变色：
   - tag: 免杀 -> 粉色系
   - tag: 渗透 -> 蓝色系
   - tag: 开源 -> 黄色系
   - tag: 其他 -> 紫色系

2. Markdown 表格会自动变成美化表格：
   - 表头使用对应分类颜色
   - 隔行浅色底
   - 保持类似你图三那种处理效果

3. Mermaid 流程图可以正常显示：
   写法：
   ```mermaid
   flowchart TD
     A[开始] --> B[信息收集]
     B --> C[API 分析]
   ```

4. 保留 v29 需求锁定版：
   - 首页三张分类卡片固定保留
   - 分类为空时卡片和猫图不删除
   - 近期更新显示最近五篇
   - 分类页 category.html 正常使用

上传方式：
覆盖 GitHub 根目录：
- index.html
- post.html
- category.html

文章格式示例：
---
title: Web / SRC 信息搜集到 API 分析学习流程
date: 2026-05-31
tag: 渗透
summary: 适用场景：授权测试、靶场环境、自有资产梳理。
---

# 标题

```bash
ksubdomain verify -f all_domains.txt -o result.txt
```

| 工具 / 平台 | 主要作用 | 你要拿到什么 |
|---|---|---|
| FOFA | 网络空间资产搜索 | URL、域名、IP、端口 |
