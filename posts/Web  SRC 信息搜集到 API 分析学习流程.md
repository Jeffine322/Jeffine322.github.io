---
title: Web SRC 信息搜集到 API 分析学习流程
date: 2026-06-01
tag: 渗透
summary: 记录 Web/SRC 信息搜集、资产整理、接口发现和 API 分析的学习流程。
cover: ./assets/cat-cry.png
---
# Web / SRC 信息搜集到 API 分析学习流程

## 1、资产收集流程

### fofaview搜索

在fofaview搜索主域名

```
domain="domain="hljedu.gov.cn""
```

导出host 作为第一批资产

还可以用别的工具继续搜索域名比如virustotal、微步、证书透明度、ip138、dnsdumpster、oneforall爆破这些工具进行补充域名

目标是得到一批属于目标单位的子域名

### ksubdomain进行验证已有域名是否能够解析

上一环节拿到的域名用Ksubdomain验证哪些是还可以解析的

```
ksubdomain.exe v -f hosts.txt -o result.txt
```

 得到result.txt之后，把它整理成纯域名列表，方便后续给OneForAll或者其他工具使用![image-20260515200941998](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260515200941998.png)

PowerShell 处理示例：

```
Get-Content "C:\Users\Jeffine\Documents\KSubdomain-v2.4.0-windows-amd64\result.txt" |
Where-Object { $_ -match "=>" } |
ForEach-Object { ($_ -split "=>")[0].Trim() } |
Sort-Object -Unique |
Set-Content "C:\Users\Jeffine\Documents\KSubdomain-v2.4.0-windows-amd64\oneforall_targets.txt"
```

### Oneforall跑公开来源收集

用整理好的域名继续跑OneForAll：

```
python oneforall.py --targets C:\Users\Jeffine\Documents\hlj\oneforall_targets.txt --alive False --brute False --fmt csv run
```

需要的话，后面进行Oneforall爆破得到目标域名



得到域名可以使用https://uutool.cn/extract-domain/进行去重和提取域名



### CDN/源站判断

去重后的域名放在Eeyes里看看有没有CDN、指纹和IP信息：

```
Eeyes.exe -f C:\Users\Jeffine\Documents\hlj\uutool.txt > C:\Users\Jeffine\Documents\hlj\eeyes_result.txt
```

找到没有cdn的ip之后对其进行整理去重（可以用powershell/uutool）得到ip 

后续对这些IP做端口和Web存活判断

### 端口发现

用得到的ip找开放端口

```
命令：
masscan -p1-65535 -iL no_cdn_ips.txt -oL hlj_ip_port.txt --rate=10000
```

### Web存活判断

将IP、域名、IP:端口放在一个txt里，使用httpx来判断web服务

```
./httpx -l ip&ip_port.txt -o web.txt
```

得到web存活列表之后，再对其进行指纹识别和人工筛选

### 指纹识别

用Finger做进一步查看具体是什么系统/技术栈。识别系统类型、框架和中间件：

```
python Finger.py -f ../hlj_ip_port.txt
```

指纹识别之后重点关注有没有熟悉的cms和框架例或者是价值比较高的指纹，比如：

- Shiro
- WordPress
- 若依
- Spring Boot
- ThinkPHP
- PHPWind
- SimpleBoot / ThinkCMF
- Swagger
- 老版本 Apache / PHP / Tomcat 等

### 批量截图和人工筛选

批量截图工具：https://github.com/yunxiaoshu/eyeurl

截图之后可以进行人工筛选一些页面重点看相应和请求如：

- 后台登录页
- 数据平台
- API 文档
- 老系统
- 管理后台
- 统一认证系统
- 上传、下载、导出、报表类页面

## 2、测网站

### 插件工具的使用

这次用到的插件有Findsomething、VueCrack、横戈团队Webpack映射提取器、雪瞳还有bp里面的HaE

用法：

#### 2.1 FindSomething

作用：从页面和 JS 里提取 URL、接口、邮箱、手机号、路径等信息。

用法：

1. 插件亮起来后点开。
2. 查看 `URL / StaticPath / API` 等结果。
3. 把有价值的 URL 复制到记事本。
4. 不要所有 URL 都请求，先分类。

分类方式：

- 静态资源：`js / css / png / jpg`
- 前端路由：`/#/login`、`/teachersDetails`
- 后端接口：`/api/xxx`、`/bi/xxx`
- 高风险动作：`delete / update / export / import / upload`

---

#### 2.2 VueCrack

作用：解析 Vue 前端里的路由、组件和页面。

适合看：

- 有哪些前端页面
- 有没有隐藏路由
- 管理员页面路径
- 某个页面对应哪个 JS 文件

注意：能看到前端路由，不代表能访问后端数据。

---

#### 2.3 Webpack 映射提取器

作用：从 Webpack 打包文件里提取映射关系、JS 文件、页面模块。

适合用来找：页面对应的 JS 文件、接口路径、前端字段名、图片字段、`rootUrl / baseURL / appSecret / sign` 这类配置

---

#### 2.4 HaE

Burp 里的 HaE 用来看响应包里的敏感信息。

使用位置：

```text
HTTP history → Response → MarkInfo
```

常见能看到：手机号、身份证格式、token、key、password、api、内网 IP、其他敏感字段

注意：HaE 可能误报。比如 WAF 的事件编号可能被识别成身份证号，需要人工确认上下文。

---

#### 2.5 eyeurl

作用：批量请求 URL / 批量截图。

项目地址：

```text
https://github.com/yunxiaoshu/eyeurl
```

注意：批量请求要控制频率，不要把危险接口也丢进去跑。

### 看URL结果

FindSomething 或其他工具提取出来的 URL，不要直接全点。先判断类型。

#### 3.1 静态资源

例如：

```text
/js/app.xxx.js
/public/js/common.js
/admin/themes/simplebootx/Public/assets/css/admin_login.css
```

这种可以直接浏览器打开，看源码内容。

重点搜：

```text
api
login
token
baseURL
rootUrl
sign
upload
download
file
path
```

---

#### 3.2 前端路由

例如：

```text
/teachersDetails
/schoolList
/courseOverview
```

这类一般只是前端页面路径，不一定是后端接口。

---

#### 3.3 后端接口

例如：

```text
/api/login
/bi/teacher/get/
/api/school/listSchool
```

这类要放 Burp 里看，不要直接批量请求。

注意前端域名和后端域名可能不同，比如：

```text
前端：https://bi.hljedu.gov.cn
后端：https://bi-api.hljedu.gov.cn
```

工具提取出来可能拼的是前端域名，测试时要结合 JS 里的 `rootUrl / baseURL` 判断真实后端。

---

#### 3.4 危险接口

看到这些关键词要先停一下：

```text
delete
remove
update
save
add
insert
reset
clear
import
export
upload
```

这类接口可能会改数据、删数据、导出数据。  

## 找源代码的一些尝试

找源码不是直接“右键查看源码”。右键看到的一般只是前端 HTML / JS / CSS。  
真正后端源码通常只有这些情况下才能看到：

- `.git` 暴露
- 备份包泄露
- 目录浏览泄露
- 任意文件读取
- 服务器只读权限
- 授权方直接给源码或部署包

### 5.1 `.git` 检查

最小化验证：

```text
/.git/HEAD
/.git/config
/.git/index
```

判断：

- `/.git/HEAD` 返回 `ref: refs/heads/master` 或 `main`：疑似 `.git` 暴露
- `/.git/config` 返回 `[core]`、`[remote]`：基本确认
- `/.git/index` 返回二进制或 `DIRC`：辅助确认
- 返回 404 / 首页 / WAF：暂未发现

不要一上来完整 dump 仓库。

---

### 5.2 备份包检查

常见路径：

```text
/www.zip
/web.zip
/backup.zip
/source.zip
/src.zip
/code.zip
/wwwroot.zip
/database.sql
/db.sql
/backup.sql
```

如果根据目录发现了特殊路径，可以定向尝试：

```text
/admin.zip
/public.zip
/simplebootx.zip
/themes.zip
/Application.zip
/Runtime.zip
/Uploads.zip
```

命中判断：

- `200 OK`
- `Content-Type: application/zip` 或 `application/octet-stream`
- `Content-Length` 明显较大
- 响应体开头是 `PK`

如果是 SQL：

- 出现 `CREATE TABLE`
- 出现 `INSERT INTO`
- 出现 `DROP TABLE`

---

### 5.3 目录浏览

可以手工看：

```text
/public/
/public/js/
/admin/
/admin/themes/
/admin/themes/simplebootx/
/admin/themes/simplebootx/Public/
```

如果返回：

```text
Index of /xxx/
```

说明目录浏览开启。  如果是 403 / 404 / 页面错误，就不算。

---

### 5.4 目录穿越和任意文件读取

这类问题不能硬测，要先找到入口。

常见入口关键词：

```text
download
upload
file
attachment
preview
path
filename
filepath
image
img
```

常见参数：

```text
file=
path=
filename=
filepath=
url=
src=
id=
```

思路：

1. 先找到正常的文件下载/预览接口。
2. 确认参数确实控制文件路径或文件 ID。
3. 只做最小化验证。
4. 不读取配置文件、私钥、数据库密码、真实敏感文件。
5. 没有入口就不要硬塞 `../`。

###  5.5 通过特殊字符串找同源源码

不要搜太通用的文件名，比如：

```text
jquery.js
ajaxForm.js
common.js
```

这些太常见，搜出来一堆没法判断。

更有效的是搜目标网站里比较特殊的字符串，例如：

- 某个特殊的 `div id`
- 某个特殊的 `class name`
- 页面标题
- `form action` 地址
- JS 变量名
- CSS / JS 注释
- 特殊目录路径
- 系统名称

例子：

```text
"海南省教育消费回流统计联网直报平台"
"admin/themes/simplebootx/Public/assets/css/admin_login.css"
"index.php?g=admin&m=public&a=dologin"
"GV = {" "DIMAUB" "JS_ROOT"
"PHPWind JS core" "ajaxfileupload" "swfupload"
```

这种方式适合：

- 找同款框架
- 找相似项目
- 找公开源码
- 做技术栈溯源

但要注意：找到同类项目，不等于就是目标站源码。  
只有系统标题、目录结构、接口路径、页面内容等多个特征都一致时，才能说“疑似同源”。

---

### 5.6  GitHub Code 搜索方法

进入 GitHub 后，在搜索框里输入关键词，回车后切到code：

常用搜索方式：

```text
"海南省教育消费回流统计联网直报平台"
"admin/themes/simplebootx/Public/assets/css/admin_login.css"
"index.php?g=admin&m=public&a=dologin"
filename:admin_login.css "phpwind.com"
filename:wind.js "PHPWind JS core"
language:PHP "g=admin" "m=public" "dologin"
```

判断匹配程度：

- 只匹配通用 JS：弱
- 匹配目录结构和路由：中
- 匹配标题、action、目录、注释：强
- 出现目标域名或系统名：非常强



## 顺了一下webtest

把得到的域名进行httpx扫描得到存活的web，然后用httpx自带信息看标题和技术栈

```
./httpx -l ../webtest/csc_alive_web_clean.txt -title -status-code -tech-detect -follow-redirects -o ../webtest/csc_httpx_finger.txt
```

把疑似重点资产用关键词筛出来

```
grep -Ei "login|admin|manage|system|api|swagger|druid|nacos|actuator|shiro|cas|sso|oauth|统一认证|登录|后台|管理|系统|平台|服务|接口|数据|Spring|Tomcat|ThinkPHP|若依|ruoyi|YApi|Jenkins|Grafana|MinIO|phpMyAdmin" ../webtest/csc_httpx_finger.txt > ../webtest/important.txt
```

得到import.txt然后跑Eeyes或者Finger

```
python3 ./Finger/Finger.py -f ../webtest/csc_alive_web.txt > ../webtest/csc_finger_result.txt
```

