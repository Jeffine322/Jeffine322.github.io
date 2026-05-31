Scholar's Notes v33

这版把文章详情页 post.html 的 Markdown 渲染改为 marked.js。

为什么要改：
你截图里 Markdown 本身是正常的，但旧版自写解析器对复杂 Markdown 兼容不够。
GitHub 能正常渲染，不代表我们自写解析器也能完全正确。
这版直接用成熟 Markdown 渲染器 marked.js，处理代码块、表格、列表会更稳。

修复内容：
1. 代码块不会把后面的正文染成深色。
2. 代码块是：浅色标题条 + 深色代码区。
3. 代码文字强制白色。
4. Markdown 表格自动转成彩色表格。
5. Mermaid 流程图继续支持。
6. 颜色继续按 tag 分类：
   免杀 粉色
   渗透 蓝色
   开源 黄色
   其他 紫色

上传方式：
覆盖仓库根目录：
- index.html
- post.html
- category.html

重点：
一定要覆盖 post.html。
