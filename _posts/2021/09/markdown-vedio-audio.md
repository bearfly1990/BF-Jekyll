---
layout: post
title: 在Markdown中使用音频与视频
subtitle: 使用HTML标签在Markdown中来引入音频与视频
date: 2021-09-09
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
header-img: /img/bf/python.jpg
tags:
  - markdown
  - audio
  - vedio
---

### Audio and Vedio in markdown

Markdown默认是支持Html标签的，所以我想到视频和音频应该也是支持的吧。

下面是简单的使用，Mark一下。
<!--more-->
### 简单的使用

#### Vedio

```html
<video id="video" controls="" preload="none" poster="/img/bf/python.jpg">
    <source id="mp4" src="/vedio/test.mp4" type="video/mp4">
</video>
```

<video id="video" controls="" preload="none" poster="/img/bf/python.jpg">
      <source id="mp4" src="/vedio/test.mp4" type="video/mp4">
      </video>

#### Audio

```html
<audio id="audio" controls="" preload="none">
    <source id="mp3" src="/audio/test.mp3">
</audio>
```

<audio id="audio" controls="" preload="none">
    <source id="mp3" src="/audio/test.mp3">
</audio>
