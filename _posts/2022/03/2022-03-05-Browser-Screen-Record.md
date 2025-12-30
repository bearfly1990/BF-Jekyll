---
layout: post
title: Browser Screen Recording
subtitle: 原生JS录制屏幕
date: 2022-03-05
author: BF
thumbnail: /img/post-bg-github-cup.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/post-bg-github-cup.jpg
tags:
    - getDisplayMedia
    - getUserMedia
---

### Overview

前两天看到一个原生 JS 用浏览器录制屏幕的方法，挺有意思的，今天试着想把声音也录制下来，看了一天，还是没有解决，网上的方法都不对。。

<!--more-->

### 核心代码

主要是利用`navigator.mediaDevices.getDisplayMedia`来实现视频录制。

尝试过使用`navigator.mediaDevices.getUserMedia`来获取音频（麦克风），但是暂时没有办法和视频合在一起。

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Recording</title>
        <meta charset="UTF-8" />
        <style>
            body {
                font-family: Arial;
                margin: 4vh auto;
                width: 90vw;
                max-width: 600px;
                text-align: center;
            }
            #controls {
                text-align: center;
            }
            .record-btn {
                margin: 10px 5px;
                padding: 15px;
                background-color: #2bcbba;
                border: none;
                color: white;
                font-weight: bold;
                border-radius: 6px;
                outline: none;
                font-size: 1.2em;
                width: 120px;
                height: 50px;
            }
            .record-btn:hover {
                background-color: #26de81;
                cursor: hand;
            }
            .record-btn:disabled {
                background-color: #2bcbba80;
            }
            #stop {
                background-color: #fc5c65;
            }
            #video {
                margin-top: 10px;
                margin-bottom: 20px;
                border: 12px solid #a5adb0;
                border-radius: 15px;
                outline: none;
                width: 100%;
                height: 400px;
                background-color: black;
            }
            h1 {
                color: #2bcbba;
                letter-spacing: -2.5px;
                line-height: 30px;
            }
            .created {
                color: lightgrey;
                letter-spacing: -0.7px;
                font-size: 1em;
                margin-top: 40px;
            }
            .created > a {
                color: #4b7bec;
                text-decoration: none;
            }
        </style>
    </head>
    <body>
        <h1><u style="color:#fc5c65">Recording</u></h1>
        <video class="video" width="600px" controls></video>
        <button class="record-btn">record</button>

        <!-- <script src="./index.js"></script> -->
    </body>
    <script>
        let btn = document.querySelector(".record-btn");

        btn.addEventListener("click", async function () {
            // let audioStream = await navigator.mediaDevices.getUserMedia({
            //     // video: { width: 1280, height: 720 },
            //     audio: true
            // });

            let videoStream = await navigator.mediaDevices.getDisplayMedia({
                video: true,
                // audio: true,   //not support
                cursor: "always",
            });

            // videoStream.addTrack(audioStream.getAudioTracks()[0]);

            const mime = MediaRecorder.isTypeSupported("video/webm; codecs=vp9")
                ? "video/webm; codecs=vp9"
                : "video/webm";
            let mediaRecorder = new MediaRecorder(videoStream, {
                mimeType: mime,
            });

            let chunks = [];
            mediaRecorder.addEventListener("dataavailable", function (e) {
                console.log("dataavailable", e.data);
                chunks.push(e.data);
            });

            mediaRecorder.addEventListener("stop", function () {
                console.log("stop");
                let blob = new Blob(chunks, {
                    type: chunks[0].type, //,"video/mp4"
                });
                let url = URL.createObjectURL(blob);

                let video = document.querySelector("video");
                video.src = url;

                let a = document.createElement("a");
                a.href = url;
                a.download = "video.webm"; //video.mp4
                a.click();
            });

            mediaRecorder.start();
        });
    </script>
</html>
```

### Demo

![API Tool Demo](/img/post/2022/03/2022-03-05-Browser-Screen-Record/recording-demo.gif)

### 参考

[用JS创建一个录屏功能](https://www.jb51.net/article/228779.htm)
[用网页来录制视频](https://www.jianshu.com/p/58447ece6fd1)
