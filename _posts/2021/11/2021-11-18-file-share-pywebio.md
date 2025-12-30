---
layout: post
title: Files sharing by pywebio
subtitle: create file sharing web application by pywebio
date: 2021-11-18
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
header-img: /img/bf/python.jpg
tags:
  - pywebio
  - python
---

## 背景

之前试过用`pyftpdlib`来使用ftp的方式来分享文件，但是发现文件比较大的时候会有问题，而且需要登录很不方便。

最近发现一个很好用的来建立web应用的库`pywebio`可以达到我想要的文件共享的效果，主要是构建真的很方便，不需要写前端，直接用`python`代码就可以了。
<!--more-->
## 安装库

[https://github.com/pywebio/PyWebIO](https://github.com/pywebio/PyWebIO)

```batch
pip3 install pywebio
```

## 代码

下面直接上代码，大家可以看到非常简洁，控件的定义也比较优雅，像一些简单的GUI界面都可以迁移过来。

```python
from pywebio.input import input, FLOAT, file_upload
from pywebio.output import put_text, put_file,close_popup, popup, put_buttons, put_markdown
from pywebio.session import set_env, hold
from pywebio import start_server
import time
import glob
import os
import pywebio.output as output
from functools import partial

def upload():
    list_files()
    files = file_upload("Upload a file", multiple=True, max_size='4G')
    for f in files:
        open(os.path.join("Shared", f['filename']), 'wb').write(
            f['content'])
        # alert(f"Upload {f['filename']} Successfully!")

def list_files():
    all_files = glob.glob("shared/**", recursive=True)
    file_output_list = []
    for filename in all_files:
        print(all_files)
        if os.path.isdir(filename):
            continue
        else:
            file_output_list.append(filename)

    with output.use_scope('files', clear=True):
        output.put_table([
            ['file', 'full name', 'action'],
            *input_file_local(file_output_list)
            ])
            

def input_file_local(filename_list):
    put_files = []
    for filename in filename_list:
        with open(filename, 'rb') as fh:
            put_files.append([put_file(name=os.path.basename(filename), 
                content=fh.read()), filename, put_buttons(['delete'], onclick=partial(edit_row, filename=filename))])
    return put_files

def edit_row(choice, filename):
    print(choice, filename)
    # os.remove(filename)
    # list_files()
    alert("Delete is disabled")

def alert(message, title='Info'):
    popup(title, [
        put_text(message),
        put_buttons(['OK'], onclick=lambda _:close_popup())
    ])

def main():
    while(True):
        upload()

start_server(main, port=8080, debug=True, max_total_size="5G", max_payload_size="5G")
```

从代码里可以看出，我们上传的文件都放在`shared`这个目录，所以目前需要手动建立，后面上传的文件都会存在这里。

这里为了支持大文件的上传，加到了`5G`。

## 效果演示

![File Shares Demo](/img/post/2021/11/2021-11-18-file-share-pywebio/file-share-demo.gif)

这里我们把删除的操作给disable了，需要的话可以加回去。不过一般还是自己控制比较好。

```python
def edit_row(choice, filename):
    os.remove(filename)
    list_files()
```

## 参考

- [PyWebIo | 快速构建web应用](https://blog.csdn.net/qq_40442753/article/details/118426603)
