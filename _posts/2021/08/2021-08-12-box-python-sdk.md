---
layout: post
title: box SDK for python
subtitle: Using Box SDK to connect box app by python
description: Using Box SDK to connect box app by python
date: 2021-08-12
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
tags:
  - python
  - box
  - sdk
---
<!--
header-img: /img/bf/python.jpg
photos: 
  - "https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png"
  -->
### Box

Box ([http://www.box.com](http://www.box.com)) 是公司在用的网盘工具，是平时保存分享资料的重要地方。最近参与的项目客户会把文件放在网盘中，给我们发送网盘路径，让我们自己去取，这里就涉及到权限的问题。今天主要试了一下用box-sdk-python的方式去上传和下载文件。

> Note:今天访问的一些资源可能需要大家自己会科学上网。
<!--more-->
### Box App 配置

最后代码的使用比较简单，主要是前面的一些配置需要知道一下。

首先要注册并登录box账号，默认的10G空间大小足够了。

进入到网盘后，点击左下脚的 `Dev Console`
![box-main](/img/post/2021/08/box-main.png)

这里可以新建`App`，到时候就是一个独立的文件存储的空间
![box-create-new-app](/img/post/2021/08/box-create-new-app.png)

选择`Custom App`
![box-create-custom-app](/img/post/2021/08/box-create-custom-app.png)

再选择`JWT`方式并输出App Name
![box-create-custom-app-jwt](/img/post/2021/08/box-create-custom-app-jwt.png)

然后在`Configuration`tab我们就能看到很多Credentials验证的信息

需要的一些权限要勾上
![](/img/post/2021/08/box-config-access.png)
主要点击下面的`Generate a Public/Private Keypair`，就会在生成密钥，并下载相关配置的json文件。

> 注意: 第一次用的时候需要你开启双重验证，需要手机下载**Authenticator** 在提示的另一个页面扫描二维码把动态口令牌导入你的手机，后面一些重要的操作都需要输入。

![box-generate-private-public-keypair](/img/post/2021/08/box-generate-private-public-keypair.png)
![box-json-config-alert](/img/post/2021/08/box-json-config-alert.png)
![box-public-key](/img/post/2021/08/box-public-key.png)
![box-json-config.png](/img/post/2021/08/box-json-config.png)

下面是json 配置文件的信息，一会儿连接的时候需要用到的信息都在里面。

```json
{
  "boxAppSettings": {
    "clientID": "r76ashyr9ehltsuq8mfbtbxml1ugzqxb",
    "clientSecret": "WK0Sp3T1Hvb8hVvFYScY2JSolLfIjHSi",
    "appAuth": {
      "publicKeyID": "4885s2em",
      "privateKey": "-----BEGIN ENCRYPTED PRIVATE KEY-----\nMIIFDjBABgkqhkiG9w0BBQ0wMzAbBgkqhkiG9w0BBQwwDgQIBQLozbR/T3oCAggA\nMBQGCCqGSIb3DQMHBAgbdNCj60pETwSCBMjTF6vr+mqyEeSoy/x9RtRQbIBUYZnT\n6OHkeIgnQAZbs4VADyWitrbasEOQSD5OLfk0TFYtN/O4RQ6pj/dirwrWYCXyvB8x\nPfZ+MjzY2vAJbJle5L7dT6Eq3eZLzbluZ6IsD+KenwTpK4iZcWqgRIjTJpKOgfOl\nmP8R0KH+vnADXllnhaAQIUnbe9fsg+sM58rciFvOGxb7slyF/xecZ8vhMH6RTDA6\nhwC+8HtF76lZ0tZ7Lqo1ZMn3tasrEBIGhFnPDzNG0lFVzxUWfyhf61WSWcirMwV5\nLqt2ZPiLl5wKlVHtAjLQw8ylfv9kDRhIlPTLUr50tdfP9rxGq2yrwjr9ksoTUw9k\nK48vcp1j2nW5ekDwmhRwYbEu/hkg4t26msuOAEKiHtv2lpiEJyORinbIWJhM6BDO\nJvr1EgAStz3WgOoVIXjm99w0LaLa8TexR+wH6bmxWR7GaEgISlxqEhDxsPfg9yZl\noH/3ab81uAxWKUU9D/JaYlc6MN5D0OKFlJZll9Zfd/yIvU5fTYoAzNC/VmKcXqWz\nGIj7Y6bzB7U9glsAVGkwlrN5ReI7SZb6pisDWvNyJcSZUsjOyKFcNwMWZccdKFy+\nMg4FT/KxCtIWP+S46kkBy0RWpxKGgqtNUi/LVbw9S9GqDDrvICJiwKKQ6EGvy+8f\nCdluDENzMBQvsY9CfoYBpbNbACSxhyEePHqitOYsVucVaTtxPyPD5CYU7ZeB5XBk\nN0Ygq33jS3Exnp3b5eEArLm14L/ajmpitizhtuRIhHkHLeuP1bXGwq2ZofD4wSY9\nQw7u6uN+v6q8qRccZODSip6WsYcNzlF7E4C8Ubk7om1sniXN3Y8cPpitp8zClWun\nnQKroK9rUyw7mVTYEse8syeL/pMvSwXnkmWtppr2mMuCZ3R2MAzXLkaIMnk0yPuf\n70J9rnYOV+l3npyeGcVz0C+rograaXMaYH3ZdSe6vIJqISfibzowCzJuZDAkJCl7\nCbt5lWEAm4dhSTwYDjdA2putodh22olgFbcsm7VMOcs3+N/eQQ501NzAYlVLFgfK\n9LLtNVySjuEOaH4xSzdnIjUIquHZMlVjivxyeG+z6EdVNUYYXapzwo66al2IDu0e\nHBF+qE59MbJyZW6l7HeLOhPtB8+LqJ46h8ngSy0clCep6VNvCAcxVGGBeVx2TrGr\nwxUs/MSgONArAB+D/zA8IR/o9+uIPkJWVvImAEc9oGNxfmzPnzpS7756gHe3+Oqt\nttgxGP0Jlf0iPvaU1NDM/cJxw0kliwOad50+9QdufuZTuvxEbejxibwLNir/U3NB\n9sRFpdvOu+U3UocmfYy8jXdGTVWczk6vk/vB9WHnARTctPLAwuLyFo2jVZgjb5u3\nDzoYH7jNseI8s/vz/M0KUTt1dhNM0/nCdkXia7CAeC9L671Z5OaTUawuyvYqsrJt\nqOfHdOUraXaE9pESThjF9G5qlqZwuQlHv2sRpwTOOxrqFWveOWGEeIC8YaGYgidi\nX3f2lNY8BozAueANEwwYIkTeU7MMQ3diCC84J1+lk93DmeIrqfHRjap9Lu6ckF2a\nYITCOdpMT23GfJcRTT2MfW5Sy5nY2S2uh2L6zJwLWi/hO5xoRWEMwhvJC91mQmMq\neJs=\n-----END ENCRYPTED PRIVATE KEY-----\n",
      "passphrase": "e95a0532aeccf28116ee3be84761297d"
    }
  },
  "enterpriseID": "836874028"
}
```
下面还有最重要的一步，需要开放这个App的访问权限，不然也还是没有办法使用。
![](/img/post/2021/08/box-app-authorization-tab.png)
![](/img/post/2021/08/box-app-authorization-submit.png)

在提交了请求之后，注册的邮件中就能收到request
![](/img/post/2021/08/box-app-authorization-email.png)
点击`Review App`之后，就跳转到Review页面，点击`Authorize`
![](/img/post/2021/08/box-app-authorization-review.png)
再回到TestDemo就是Enable的状态了。
![](/img/post/2021/08/box-app-authorization-enabled.png)

### Box SDK Python

Box为各种语言都提供了访问的SDK API，python的话有[box-python-sdk](https://github.com/box/box-python-sdk)

这里有文档详细的说明了怎么操作网盘
[https://github.com/box/box-python-sdk/tree/main/docs/usage](https://github.com/box/box-python-sdk/tree/main/docs/usage)

下面是主要的简单演示:

我们首先要把json中的rsa private key单独放`.pem`文件中e.g.rsa_private_key.cxdemo.pem

> 注意：这里需要把json文件中的\n字符串替换成真正的换行符。

如下：
```
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFDjBABgkqhkiG9w0BBQ0wMzAbBgkqhkiG9w0BBQwwDgQIBQLozbR/T3oCAggA
MBQGCCqGSIb3DQMHBAgbdNCj60pETwSCBMjTF6vr+mqyEeSoy/x9RtRQbIBUYZnT
6OHkeIgnQAZbs4VADyWitrbasEOQSD5OLfk0TFYtN/O4RQ6pj/dirwrWYCXyvB8x
PfZ+MjzY2vAJbJle5L7dT6Eq3eZLzbluZ6IsD+KenwTpK4iZcWqgRIjTJpKOgfOl
mP8R0KH+vnADXllnhaAQIUnbe9fsg+sM58rciFvOGxb7slyF/xecZ8vhMH6RTDA6
hwC+8HtF76lZ0tZ7Lqo1ZMn3tasrEBIGhFnPDzNG0lFVzxUWfyhf61WSWcirMwV5
Lqt2ZPiLl5wKlVHtAjLQw8ylfv9kDRhIlPTLUr50tdfP9rxGq2yrwjr9ksoTUw9k
K48vcp1j2nW5ekDwmhRwYbEu/hkg4t26msuOAEKiHtv2lpiEJyORinbIWJhM6BDO
Jvr1EgAStz3WgOoVIXjm99w0LaLa8TexR+wH6bmxWR7GaEgISlxqEhDxsPfg9yZl
oH/3ab81uAxWKUU9D/JaYlc6MN5D0OKFlJZll9Zfd/yIvU5fTYoAzNC/VmKcXqWz
GIj7Y6bzB7U9glsAVGkwlrN5ReI7SZb6pisDWvNyJcSZUsjOyKFcNwMWZccdKFy+
Mg4FT/KxCtIWP+S46kkBy0RWpxKGgqtNUi/LVbw9S9GqDDrvICJiwKKQ6EGvy+8f
CdluDENzMBQvsY9CfoYBpbNbACSxhyEePHqitOYsVucVaTtxPyPD5CYU7ZeB5XBk
N0Ygq33jS3Exnp3b5eEArLm14L/ajmpitizhtuRIhHkHLeuP1bXGwq2ZofD4wSY9
Qw7u6uN+v6q8qRccZODSip6WsYcNzlF7E4C8Ubk7om1sniXN3Y8cPpitp8zClWun
nQKroK9rUyw7mVTYEse8syeL/pMvSwXnkmWtppr2mMuCZ3R2MAzXLkaIMnk0yPuf
70J9rnYOV+l3npyeGcVz0C+rograaXMaYH3ZdSe6vIJqISfibzowCzJuZDAkJCl7
Cbt5lWEAm4dhSTwYDjdA2putodh22olgFbcsm7VMOcs3+N/eQQ501NzAYlVLFgfK
9LLtNVySjuEOaH4xSzdnIjUIquHZMlVjivxyeG+z6EdVNUYYXapzwo66al2IDu0e
HBF+qE59MbJyZW6l7HeLOhPtB8+LqJ46h8ngSy0clCep6VNvCAcxVGGBeVx2TrGr
wxUs/MSgONArAB+D/zA8IR/o9+uIPkJWVvImAEc9oGNxfmzPnzpS7756gHe3+Oqt
ttgxGP0Jlf0iPvaU1NDM/cJxw0kliwOad50+9QdufuZTuvxEbejxibwLNir/U3NB
9sRFpdvOu+U3UocmfYy8jXdGTVWczk6vk/vB9WHnARTctPLAwuLyFo2jVZgjb5u3
DzoYH7jNseI8s/vz/M0KUTt1dhNM0/nCdkXia7CAeC9L671Z5OaTUawuyvYqsrJt
qOfHdOUraXaE9pESThjF9G5qlqZwuQlHv2sRpwTOOxrqFWveOWGEeIC8YaGYgidi
X3f2lNY8BozAueANEwwYIkTeU7MMQ3diCC84J1+lk93DmeIrqfHRjap9Lu6ckF2a
YITCOdpMT23GfJcRTT2MfW5Sy5nY2S2uh2L6zJwLWi/hO5xoRWEMwhvJC91mQmMq
eJs=
-----END ENCRYPTED PRIVATE KEY-----
```

然后读取json中的配置，创建client.
```python
import json
from boxsdk import JWTAuth
from boxsdk import Client

json_file = open ('836874028_4885s2em_config.json', "r")
config = json.load(json_file)

#param2不能少，这里可以拿到临时的token，虽然这里我们用不到。
def your_store_tokens_callback_method(token, param2):
    print(token)

auth = JWTAuth(
    client_id=config['boxAppSettings']['clientID'],
    client_secret=config['boxAppSettings']['clientSecret'],
    enterprise_id=config['enterpriseID'],
    jwt_key_id=config['boxAppSettings']['appAuth']['publicKeyID'],
    rsa_private_key_file_sys_path='rsa_private_key.cxdemo.pem',
    rsa_private_key_passphrase=config['boxAppSettings']['appAuth']['passphrase'],
    store_tokens=your_store_tokens_callback_method,
)

access_token = auth.authenticate_instance()
client = Client(auth)
```

使用`client`就可以做许多操作了。
```python
root_folder = client.root_folder().get()
subfolder = root_folder.create_subfolder('Test')
print('Created subfolder with ID {0}'.format(subfolder.id))
# subfolder = client.folder('0').create_subfolder('Test')
subfolder = root_folder.create_subfolder('Test2')
print('Created subfolder with ID {0}'.format(subfolder.id))

subfolder = root_folder.create_subfolder('Test3')
print('Created subfolder with ID {0}'.format(subfolder.id))

client.folder(folder_id=f'{subfolder.id}').delete()

uploade_file = client.folder('0').upload('./test.txt')
print('File "{0}" uploaded to Box with file ID {1}'.format(uploade_file.name, uploade_file.id))

# Write the Box file contents to disk
output_file = open('./download.txt', 'wb')
client.file(f'{uploade_file.id}').download_to(output_file)

# items = client.folder(folder_id='0').get_items()
items = root_folder.get_items()
for item in items:
    print(f'{item.type.capitalize()} {item.id} is named "{item.name}"')
```

最后，我们可以进入到`Admin Console`-`Content`中去看我们App中的文件与目录
![](/img/post/2021/08/box-admin-console-content.png)


