---
layout: post
title: Record Video by Python
subtitle: Record video to mp4 and gif 
date: 2022-04-01
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/bf/python.jpg
tags:
    - python
    - gif
    - moviepy
    - Pillow
    - keyboard
---

## Overview

在工作的过程中，截图是很常用的用来描述问题的方式。

而使用`gif`这种动态图片就能像视频一样，很方便地展示操作的过程。

网上有许多录制`gif`的工具，有些视频录制软件也支持转成`gif`.

我最喜欢的一款就是神器[ScreenToGif](https://github.com/NickeManarin/ScreenToGif) 超级超级超级好用。
<!--more-->

但有的时候你可能没有办法软件来录制视频和GIF，这个时候如果你有python的环境，就可以自己写一个录制视频的小脚本了。

## 基本思路

视频和动态图片的本质就是帧(frame)的集合，那如果我们能把屏幕的连续截图合在一起，那就可以实现我的录制的想法。

所以这次用到了图片和视频处理相关的库`Pillow`, `imageio`, `moviepy`等等,为了支持快捷键，还使用了`keyboard`这个库，很好用。

```bash
certifi==2021.10.8
cffi==1.15.0
charset-normalizer==2.0.12
colorama==0.4.4
decorator==4.4.2
idna==3.3
imageio==2.16.1
imageio-ffmpeg==0.4.5
keyboard==0.13.5
moviepy==1.0.3
numpy==1.22.3
opencv-python==4.5.5.64
Pillow==9.0.1
proglog==0.1.9
pycparser==2.21
requests==2.27.1
tqdm==4.63.1
urllib3==1.26.9
```

## 核心代码

### 设置默认参数与热键

在入口函数这边，我们可以设计默认值，也可以在跑脚本的时候在命令行中传入，例如`python record_video_mp4_gif.py --gif 0`

而在最后，使用`keyboard`来将快捷键`alt+f9`和开始录制绑定。

```python
if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--savename', type=str, default=f'output.{time_flag()}.mp4', help='save file name')
    parser.add_argument('--fps', type=int, default=20, help='frame per second')
    parser.add_argument('--screen_type', default=1, type=int, choices=[0, 1],
                        help='1: generate gif, 0: not generate gif')
    parser.add_argument('--gif', default=1, type=int, choices=[0, 1],
                        help='1: generate gif, 0: not generate gif')
    args = parser.parse_args()
    print('#' * 80)
    print('press alt+F9 to start the recording')
    keyboard.add_hotkey('alt + f9', start_record, args=(args, None))
    keyboard.wait('esc')
```

### 选择录制的屏幕区域

大部分情况下我们不需要全屏录制，所以需要在屏幕中选择一个矩形的区域。

而这里的原理就是把当前屏幕截下来之后，通过画矩形的方式，将左上角和右下角的坐标记录下来。

```python
def on_mouse(event, x, y, flags, param):
    global img, point1, point2
    img2 = img.copy()
    if event == cv2.EVENT_LBUTTONDOWN:
        point1 = (x, y)
        cv2.circle(img2, point1, 10, (0, 255, 0), thickness=2)
        cv2.imshow('image', img2)
    elif event == cv2.EVENT_MOUSEMOVE and (flags & cv2.EVENT_FLAG_LBUTTON):
        cv2.rectangle(img2, point1, (x, y), (255, 0, 0), thickness=2)
        cv2.imshow('image', img2)
    elif event == cv2.EVENT_LBUTTONUP:
        point2 = (x, y)
        cv2.rectangle(img2, point1, point2, (0, 0, 255), thickness=2)
        cv2.imshow('image', img2)


def select_roi(frame):
    global img, point1, point2
    img = cv2.cvtColor(np.array(frame), cv2.COLOR_RGB2BGR)
    win_name = 'image'
    cv2.namedWindow(win_name, cv2.WINDOW_NORMAL)
    cv2.setWindowProperty(win_name, cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)
    cv2.setMouseCallback(win_name, on_mouse)
    cv2.imshow(win_name, img)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
    return point1, point2
```

### 开始录制

根据不同的录制类型，决定录制区域。

然后开始循环截取屏幕。

结束的方式则是循环中监测`alt+10`。

最后会生成`mp4`，根据需要来生成`gif`。

```python
def start_record(*args):
    args = args[0]
    print('press alt+F10 to stop the recording')
    print('#' * 80)
    savename = args.savename
    print(savename)
    cur_screen = ImageGrab.grab()
    if args.screen_type:
        height, width = cur_screen.size
        min_x, min_y, max_x, max_y = 0, 0, width, height
        fourcc = cv2.VideoWriter_fourcc(*'mp4v')
        video = cv2.VideoWriter(savename, fourcc, args.fps, (height, width))
    else:
        point1, point2 = select_roi(cur_screen)
        min_x = min(point1[0], point2[0])
        min_y = min(point1[1], point2[1])
        max_x = max(point1[0], point2[0])
        max_y = max(point1[1], point2[1])

        height, width = max_y - min_y, max_x - min_x

        fourcc = cv2.VideoWriter_fourcc(*'mp4v')
        video = cv2.VideoWriter(savename, fourcc, args.fps, (width, height))

    print(datetime.now(), '"Record Video Started..."')
    imageNum = 0
    while True:
        imageNum += 1
        captureImage = ImageGrab.grab()
        frame = cv2.cvtColor(np.array(captureImage), cv2.COLOR_RGB2BGR)
        if args.screen_type == 0:
            frame = frame[min_y:max_y, min_x:max_x, :]
        video.write(frame)
        if keyboard.is_pressed('alt+f10'):
            break

    video.release()
    cv2.destroyAllWindows()
    if args.gif:
        print('save to gif...')
        videoclip = VideoFileClip(savename)
        # videoclip = videoclip.resize(0.8)
        videoclip.write_gif(f"{savename}.gif", fuzz=1, fps=videoclip.fps, loop=0, program='ffmpeg')
    print(datetime.now(), '"Record Video Ended..."')
```

### 最后输出的文件

因为在默认文件名中加了`flag`，所以会自动生成不同的文件。

而现在有一个最大的问题是，生成的`gif`是从`mp4`转来的，文件有点大，我看过一些方法，但好像都不太好。

![record-video-output](/img/post/2022/04/2022-04-01-record-video-mp4-gif/record-video-output.png)

### 完整代码

[record_video_mp4_gif.py](/document/record_video_mp4_gif.py)

## 参考

[python实现录制全屏和选择区域录屏功能](https://www.cainiaojc.com/note/qahexz.html)
