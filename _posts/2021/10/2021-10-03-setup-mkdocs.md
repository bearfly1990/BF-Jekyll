---
layout: post
title: Set up markdown wiki with mkdocs
subtitle: 使用mkdocs搭建wiki
date: 2021-10-03
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
header-img: /img/bf/python.jpg
tags:
  - markdown
  - mkdocs
  - python
---

# mkdocs

[mkdocs](https://www.mkdocs.org/) 是一个基于python的第三方库，[MkDocs中文文档](https://mkdocs.zimoapps.com/)

> MkDocs是一个快速、简单、华丽的静态网站生成器，适用于构建项目文档。文档源文件以Markdown编写，并使用一个YAML文件来进行配置。

<!--more-->
## 基础安装使用

### 安装主库

```bash
pip install mkdocs
```

## 安装 mkdocs-material 主题

`mkdocs`有自带`mkdocs`和`readthedocs`两个主题，个人比较喜欢`material`这个主题。

`pymdown-extersions`增加了许多markdown的一些功能，比如一些图标。

```bash
pip install mkdocs-material
pip install pymdown-extensions
```

## 创建项目

下面是`mkdocs`内置的几个常用命令。

```bash
mkdocs new [dir-name] - Create a new project.
mkdocs serve - Start the live-reloading docs server.
mkdocs build - Build the documentation site.
mkdocs -h - Print help message and exit.
```

创建项目就可以使用`mkdocs new mywiki`，会新建一个目录:

![mywiki folder structure](/img/post/2021/10/setup-mkdocs/mywiki-folder-structure.png)

`docs`目录存放的便是原始的我们编写的`markdown`文件

`mkdocs.yml`是mkdocs的主配置文件。

## 编辑文档

试着直接在`docs`下编辑文件

![mywiki-test-doc](/img/post/2021/10/setup-mkdocs/mywiki-test-doc.png)

然后启动服务.

```bash
mkdocs serve
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.10 seconds
INFO     -  [16:51:31] Serving on http://127.0.0.1:8000/
```

打开链接，可以看到`mkdocs`会根据目录结构自动生成导航，最里面的标题和文档的一级Header是一致的。

![default mkdocs test](/img/post/2021/10/setup-mkdocs/defult-mkdocs-test-ui.gif)

## mkdocs.yml

像上面的导航，除了自动生成，也可以在`mkdocs.yml`里自定义配置。

```yml
site_name: My Docs
site_url: https://example.com/
nav:
    - Home: index.md
    - MyPage: 
        - TestPage: folder/file.md
```

![user defined nav](/img/post/2021/10/setup-mkdocs/user-defined-nav.png)

不过个人觉得还是自动生成方便，除非需要对特别的一些页面进行配置。

下面是我用的mkdocs.yml，也是从网上拷过来改的，主要支持了`github`自动发布。

```yml
site_name: BFWiki
site_url: http://localhost:8080/BFWiki
repo_url: https://github.com/bearfly1990/BF-Wiki/tree/gh-pages
site_author: bearfly1990
site_description: wiki for bearfly1990
# copyright:
# nav:
#     - Home: index.md
#     - About: about.md
#     - Kafka:
#       - Learn Note01: 2021/08/2021-08-09-kafka-learn-01.md
# pages:
# - [index.md, Home]
# - [about.md, About]
theme:
    # name: readthedocs
    name: material
    language: 'en'
    # logo: img/xxx.ico
    # favicon: img/facicon.ico/
    primary: "Blue Grey"
    # accent: "Pink"
    features:
        # - tabs: true
        # - toc.integrate
        # - navigation.tracking
        - navigation.tabs
        - navigation.top
    highlightjs: true
    hljs_languages:
        - yaml
        - rust
    # nav_style: dark

# extra:
#     search:
#         language: 'zh'

markdown_extensions:
    - admonition
    - codehilite:
          guess_lang: false
          linenums: false
    - toc:
          permalink: "#"
    - footnotes
    - meta
    - def_list
    - pymdownx.arithmatex
    - pymdownx.betterem:
          smart_enable: all
    - pymdownx.caret
    - pymdownx.critic
    - pymdownx.details
    - pymdownx.emoji:
          emoji_generator: !!python/name:pymdownx.emoji.to_png
    - pymdownx.inlinehilite
    - pymdownx.magiclink
    - pymdownx.mark
    - pymdownx.smartsymbols
    - pymdownx.superfences
    - pymdownx.tasklist
    - pymdownx.tilde
    - pymdownx.highlight
# site_name: JetBot
# theme:
#     name: "material"
#     logo: images/logo.png
#     favicon: images/favicon.png
#     font: Incosolata
#     palette:
#         scheme: nvgreen
#     features:
#        - navigation.expand
#
# repo_url: https://github.com/NVIDIA-AI-IOT/jetbot
#
# plugins:
#   - search
# use_directory_urls: false
#
# edit_uri: blob/master/docs
# markdown_extensions:
#   - pymdownx.tabbed
#   - pymdownx.keys
#   - pymdownx.snippets
#   - pymdownx.inlinehilite
#   - pymdownx.highlight:
#         use_pygments: true
#   - admonition
#   - pymdownx.details
#   - pymdownx.superfences
#   - attr_list  # for image sizes https://github.com/mkdocs/mkdocs/issues/1678
# # use_directory_urls - False to fix broken raw html image links
# # https://github.com/mkdocs/mkdocs/issues/991
#
#
# nav:
#
#   - Home: index.md
#   - Getting Started: getting_started.md
#   - Bill of Materials: bill_of_materials.md
#   - Hardware Setup: hardware_setup.md
#   - Software Setup:
#       - Using SD Card Image: software_setup/sd_card.md
#       - Using Docker Container: software_setup/docker.md
#   - Examples:
#       - Basic Motion: examples/basic_motion.md
#       - Teleoperation: examples/teleoperation.md
#       - Collision Avoidance: examples/collision_avoidance.md
#       - Road Following: examples/road_following.md
#       - Object Following: examples/object_following.md
#   - Reference:
#       - Third Party Kits: third_party_kits.md
#       - 3D Printing: 3d_printing.md
#       - Contributing: CONTRIBUTING.md
#       - Changes: CHANGELOG.md
#       - Wi-Fi setup: software_setup/wifi_setup.md
#       - Docker Tips: reference/docker_tips.md
#
# extra_css:
#   - css/version-select.css
#   - css/colors.css
# extra_javascript:
#   - js/version-select.js
#
# google_analytics:
#   - UA-135919510-2
#   - auto
```

对了，如果想要支持中文搜索，在主题配置这边加上语言就行了。

```yml
theme:
    # name: readthedocs
    name: material
    language: 'zh'
```

## Build

上面提到的链接访问是通过本地起的python http服务，会动态重新加载每次更新后的文档。如果想要把网站部署到服务器上，就需要先编译成静态网站(html/js/css)。

操作很简单，在项目目录下执行一下`mkdocs build`就可以，会在项目下生成`site`目录，这就是编译后的结果。

![mkdocs-build](/img/post/2021/10/setup-mkdocs/mkdocs-build.png)

## Deploy To Github

另外，如果你配置过本地的`github ssh`登录，那你执行下面的命令，就能很方便的在github io上发布你的文档。

```bash
mkdocs gh-deploy
```

比如我的，他会在项目本身的repository - <https://github.com/bearfly1990/BF-Wiki/>
建立一个新的branch - [gh-pages](https://github.com/bearfly1990/BF-Wiki/tree/gh-pages), 并把编译后的静态文件上传上去。

接着就可以在<https://bearfly1990.github.io/BF-Wiki/>看到我的文档。

![my-github-wiki](/img/post/2021/10/setup-mkdocs/my-github-wiki.png)
