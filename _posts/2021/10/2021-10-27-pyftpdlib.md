---
layout: post
title: Set up ftp server with pyftpdlib
subtitle: 使用pyftpdlib搭建简易ftp server
date: 2021-10-27
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
header-img: /img/bf/python.jpg
tags:
  - ftp
  - pyftpdlib
  - python
---

## 背景

公司内部现在少了很多共享的路径，分享文件就很不方便。

在家的时候，想把文件从手机上传到电脑，也不方便（华为共享算一种方法）。

之前用python自带的http服务(`python -m http.server 8080`)可以方便分享文件给其他人，但是不能上传。

原来想着说写个页面支持上传文件，一直没有弄（虽然Alpha Test Platform支持，但没有更直接的）。

这两天想到了如果能实现简单的FTP服务，这样就方便文件交互了。试了一些方法，最后用`pyftpdlib`。

<!--more-->
## 基础安装使用

### 安装pyftpdlib

```bash
pip install pyftpdlib
```

## 直接使用命令启动

```bash
python -m pyftpdlib
```

一个简单的FTP服务器已经搭建完成，访问 `ftp://127.0.0.1:2121` 即可, 共享的路径是用户目录。

（默认IP为 127.0.0.1 、端口为 2121 ）

## 编写简易代码启动

下面是`mkdocs`内置的几个常用命令。

```python
from pyftpdlib.authorizers import DummyAuthorizer
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer
 
authorizer = DummyAuthorizer()
# username, password, shared folder, permission for files
authorizer.add_user('test', 'test', '.', perm='elradfmwMT')
# authorizer.add_anonymous('/home/nobody')
 
handler = FTPHandler
handler.authorizer = authorizer
 
server = FTPServer(('0.0.0.0', 21), handler)

server.serve_forever()
```

## 访问ftp

ftp 启动之后，在windows文件夹浏览，就可以通过 `ftp://pc-cx/`访问了（pc-cx是启动服务的机器的名字，可以用IP，默认端口是21所以可以不用写）。

这个时候需要输入上面代码中设定的用户名(test)和密码(test)，就可以看到文件，并上传和下载。
![ftp-username-password](/img/post/2021/10/pyftpdlib/username-password.png)

在手机上，可以安装ftp客户端，然后输入配置参数，连接成功后就可以与电脑文件进行交互。

## 文件权限设置

下面这句代码设置了用户名密码，ftp的路径，还有权限(perm)

```python
authorizer.add_user('test', 'test', '.', perm='elradfmwMT')
perm权限选项
读取权限：

"e" =更改目录（CWD，CDUP命令）
"l" =列表文件（LIST，NLST，STAT，MLSD，MLST，SIZE命令）
"r" =从服务器检索文件（RETR命令）
###写入权限：

"a" =将数据追加到现有文件（APPE命令）
"d" =删除文件或目录（DELE，RMD命令）
"f" =重命名文件或目录（RNFR，RNTO命令）
"m" =创建目录（MKD命令）
"w" =将文件存储到服务器（STOR，STOU命令）
"M"=更改文件模式/权限（SITE CHMOD命令）
"T"=更改文件修改时间（SITE MFMT命令）

```

## 参考

* [Pyftpdlib文档](http://pyftpdlib.readthedocs.io/en/latest/index.html)
* [Pyftpdlib 使用方法](https://blog.csdn.net/xuq09/article/details/84936853)
