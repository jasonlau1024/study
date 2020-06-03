### 安装 ConEmu

#### ConEmu 简述

​	Windows 下的终端软件很 实际上最合适做黑客的终端莫过于 PentestBox 。它不仅 

仅是一款强大的终端应用，而且还包含了很多黑客软件 其中就包含了最常用的 Metasploit 

框架。 PentestBox 使用起来虽然方便 ，却少 DIY 的乐趣和成就感。最终只选择了 

PentestBox 中使用的 终端 Con Em 。以后再慢慢 添加其他的黑客软件。

​	ConEmu 本身就是 款强大的终端。它默认将 Windows 自带的 md Powershell 

合到了一 ，稍 熟悉其功能后，就可 以配 出非常强大 终端工具。

#### ConEmu 配置

1）下载

官网：<https://conemu.github.io/>

选择稳定免安装版：Statble Portable

2）配置

`ConEmu64.exe` 64位系统

a. 添加命令补全插件

按照`ConEmu\clink\Readme.txt` 文件提示下载`clink`的`Portable`版。

加压`clink_0.4.9.zip`并复制所有文件到`ConEmu\clink`目录下。

运行`ConEmu64.exe`输入`di`测试命令补全。

### 安装 Python

> 官网：<https://www.python.org/>

#### 安装 Python2

下载地址：https://www.python.org/ftp/python/2.7.13/python-2.7.13.amd64.msi



#### 安装 Python3



#### ConEmu 配置环境变量

setting-->startup-->environment

```shell
set PATH=%PATH%;C:\Python27
set PATH=%PATH%;C:\Python38
```

保存重新打开。

测试：

```shell
python
exit();
python3
exit();
```



### 安装虚拟机

VMware

VirtualBox

Parallels

MacOS

注：Windows 10 Docker 采用 Hyper 拟机 Hyper-V 拟机和 VMware 拟机是不 共存 。在需要虚 Windows 系统 时，只有关闭 Docker 取消 Hyper 服务启系统后才能使用 VMware。



## 扫描漏洞

### 系统扫描

Vulnerability Scanner

#### 系统漏洞

不管哪个操作系统都会不时地发布补丁 这是因为操作系统在用户使用过程中不断地被发现漏洞。为了堵住这些漏洞，只有不断地打上补丁。系统漏洞造成的后果是非常严重的，其他的 程序服务漏洞还需要 间接地利用 破坏，而利 用系统漏洞大都可 直接对系统造成破坏。

#### 工具选择

***Nessus：***

老牌安全公司Tenable旗下商业收费软件，功能强大，兼容Windows和Linux。

***OpenVAS：***

开源软件，服务端只能运行在Linux上。

***Nexpose：***

Rapid7出品的商业软件，社区版可免费试用一年。

#### 安装 Nexpose(Windows)

a. 下载









