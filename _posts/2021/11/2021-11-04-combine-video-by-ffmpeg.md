---
layout: post
title: Combine Videos by ffmpeg
subtitle: using ffmpeg cmd to cut and combine videos
date: 2021-11-04
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Knowledge
header-img: /img/bf/python.jpg
tags:
  - ffmpeg
  - video
  - python
---

## 背景

之前写过文章[Cut and Combine Tiktok Videos](https://bearfly1990.github.io/2021/02/09/2021/02/2021-02-09-CutCombineTiktok/)

使用的是Python的一个第三方库来`moviepy`来操作剪辑合并视频。

最近发现一个比较好用的视频处理组件`ffmpeg` - <http://ffmpeg.org/>，也可以实现一样的效果，并且试过同样6个并发的时候，performance好很多。

```bash
ffmpeg -i input.mp4 output.avi
```
<!--more-->
## 下载ffmpeg组件

从官网下载Windows的编译好的版本，这次我们就只用到其中两个。

- `ffmpeg.exe`
- `ffplay.exe`
- `ffprobe.exe`

## 分析

因为视频的信息与处理都是通过命令行，所以定义一个`Video`类来储存基本信息。

```python
class Vedio(object):
    def __init__(self, file_name):
        vedio_attr = self.get_vedio_attribute(file_name)
        self.file_name = file_name
        self.duration = float(vedio_attr['format']['duration'])
        self.width = float(vedio_attr['streams'][0]['width'])
        self.height = float(vedio_attr['streams'][0]['height'])
        self.temp_file = ""
        self.start_time = 0
        self.ended_time = 0

    def cut_video(self, width=300, height=300):
        cmd = f'ffmpeg -ss 00:00:00 -t {self.ended_time} -i {self.file_name} -vf "scale={width}:{height}:force_original_aspect_ratio=decrease,pad={width}:{height}:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -crf 23 -c:a copy -bsf:v h264_mp4toannexb -f mpegts {self.temp_file}.ts -y'
        print(cmd)
        os.system(cmd)

    def get_vedio_attribute(self, file):
        cmd = f'ffprobe -select_streams v -show_entries format=duration,size,bit_rate,filename -show_streams -v quiet -of csv="p=0" -of json -i {file}'
        p = subprocess.Popen(cmd, stdout=subprocess.PIPE,
                             stderr=subprocess.STDOUT)
        p.wait()
        strout, strerr = p.communicate()
        attr_json = json.loads(strout)
        return attr_json
```

在这里，为了剪切后的视频能自适应新的视频大小，所以使用了pad这个过滤器,自动计算大小。

```cmd
-vf "scale={width}:{height}:force_original_aspect_ratio=decrease,pad={width}:{height}:(ow-iw)/2:(oh-ih)/2"
```

`get_vedio_attribute`返回的是视频的一些基础信息,今天就只用到了视频的总时间(`duration`)和长(`height`)宽(`width`)

```json
{
    "streams": [
        {
            "index": 0,
            "codec_name": "h264",
            "codec_long_name": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
            "profile": "High",
            "codec_type": "video",
            "codec_tag_string": "avc1",
            "codec_tag": "0x31637661",
            "width": 1280,
            "height": 1280,
            "coded_width": 1280,
            "coded_height": 1280,
            "closed_captions": 0,
            "film_grain": 0,
            "has_b_frames": 2,
            "sample_aspect_ratio": "1:1",
            "display_aspect_ratio": "1:1",
            "pix_fmt": "yuv420p",
            "level": 40,
            "color_range": "tv",
            "color_space": "bt709",
            "color_transfer": "bt709",
            "color_primaries": "bt709",
            "chroma_location": "left",
            "field_order": "progressive",
            "refs": 1,
            "is_avc": "true",
            "nal_length_size": "4",
            "id": "0x1",
            "r_frame_rate": "30/1",
            "avg_frame_rate": "30/1",
            "time_base": "1/90000",
            "start_pts": 4140,
            "start_time": "0.046000",
            "duration_ts": 8937000,
            "duration": "99.300000",
            "bit_rate": "1225216",
            "bits_per_raw_sample": "8",
            "nb_frames": "2979",
            "disposition": {
                "default": 1,
                "dub": 0,
                "original": 0,
                "comment": 0,
                "lyrics": 0,
                "karaoke": 0,
                "forced": 0,
                "hearing_impaired": 0,
                "visual_impaired": 0,
                "clean_effects": 0,
                "attached_pic": 0,
                "timed_thumbnails": 0,
                "captions": 0,
                "descriptions": 0,
                "metadata": 0,
                "dependent": 0,
                "still_image": 0
            },
            "tags": {
                "language": "und",
                "handler_name": "VideoHandler",
                "vendor_id": "[0][0][0][0]"
            }
        }
    ],
    "format": {
        "filename": "output.mp4",
        "duration": "99.347000",
        "size": "16912977",
        "bit_rate": "1361931"
    }
}
```

## 完整的代码

主体思路就是遍历当前目录中的mp4文件，根据所有视频最大的长和宽，每个视频都生成一个临时ts文件，然后再把ts文件合并在一个mp4中。

后面顺便生成mp3可以到时做铃声。

```python
import shutil
import os
import glob
from concurrent.futures import ThreadPoolExecutor, wait, ALL_COMPLETED, FIRST_COMPLETED
from datetime import datetime
import subprocess
import json
import traceback
import pathlib
"""
author: bearfly1990
create at: 02/01/2021
description:
    Utils for send email
Change log:
Date          Author      Version    Description
02/17/2021    bearfly1990 1.0.1      update combine video hight/width resize logic to adapt all videos.
02/21/2021    bearfly1990 1.0.2      Get max hight/max width from all the input videos, not hard code
11/01/2021    bearfly1990 1.0.3      Using ffmpeg/ffprobe to deal with the image
"""

# def hcf(x, y):
#     if x > y:
#         smaller = y
#     else:
#         smaller = x
#     for i in range(1,smaller + 1):
#         if((x % i == 0) and (y % i == 0)):
#             hcf = i
#     return hcf


class Vedio(object):
    def __init__(self, file_name):
        vedio_attr = self.get_vedio_attribute(file_name)
        self.file_name = file_name
        self.duration = float(vedio_attr['format']['duration'])
        self.width = float(vedio_attr['streams'][0]['width'])
        self.height = float(vedio_attr['streams'][0]['height'])
        self.temp_file = ""
        self.start_time = 0
        self.ended_time = 0

    def cut_video(self, width=300, height=300):
        # cmd_cut = f"ffmpeg -ss 00:00:00 -t {time_period} -i {input_file} -s {width}x{height} -codec copy {output_file}"
        # cmd = f'ffmpeg -ss 00:00:00 -t {time_period} -i {input_file} -vf "scale=1920:-2" -c:v libx264 -crf 1 -c:a copy {output_file}'
        # cmd = f'ffmpeg -ss 00:00:00 -t {self.ended_time} -i {self.file_name} -vf "scale={width}:{height}:force_original_aspect_ratio=decrease,pad={width}:{height}:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -crf 23 -c:a copy {self.temp_file} -y'
        cmd = f'ffmpeg -ss 00:00:00 -t {self.ended_time} -i {self.file_name} -vf "scale={width}:{height}:force_original_aspect_ratio=decrease,pad={width}:{height}:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -crf 23 -c:a copy -bsf:v h264_mp4toannexb -f mpegts {self.temp_file}.ts -y'
        
        # -s {width_max}*{height_max}  ,setsar={width_rate}:{height_rate}
        # cmd = fr'ffmpeg -ss 00:00:00 -t 15.509000000000002 -i .\test.mp4 -vf "scale=300:300" -c:v libx264 -crf 1 -c:a copy ./temp\.\test1111.mp4'
        print(cmd)
        os.system(cmd)
        # p = subprocess.Popen(cmd,stdout = subprocess.PIPE, stderr = subprocess.STDOUT)
        # p.wait()

    def get_vedio_attribute(self, file):
        cmd = f'ffprobe -select_streams v -show_entries format=duration,size,bit_rate,filename -show_streams -v quiet -of csv="p=0" -of json -i {file}'
        # result = subprocess.Popen(["ffprobe", "video.mp4"],stdout = subprocess.PIPE, stderr = subprocess.STDOUT)
        p = subprocess.Popen(cmd, stdout=subprocess.PIPE,
                             stderr=subprocess.STDOUT)
        p.wait()
        strout, strerr = p.communicate()
        attr_json = json.loads(strout)
        return attr_json


class TiktokUtil(object):
    output_folder = './output'
    temp_folder = './temp'
    max_workers = 6

    def __init__(self, remove_watermark=True, input_folder='', output_name='combined.mp4'):
        self.remove_watermark = remove_watermark
        self.output_name = output_name
        self.input_folder = input_folder
        self.video_list = []
        self.max_x_list = []
        self.max_y_list = []

    def cut_video(self, video: Vedio):
        self.max_x = max(self.max_x_list)
        self.max_y = max(self.max_y_list)
        video.cut_video(self.max_x, self.max_y)

    def call_combine_video(self, list_file="list.txt", output_file="output.mp4"):
        # cmd = f"ffmpeg -safe 0 -f concat -i {list_file} -c copy {output_file}"
        concat_files = '|'.join([f'temp\{video.file_name}.ts' for video in self.video_list])
        cmd = f'ffmpeg -i "concat:{concat_files}" -c copy -bsf:a aac_adtstoasc -movflags +faststart output.mp4 -y'
        print(cmd)
        os.system(cmd)

    def transfer_video_to_audio(self, input_file="output.mp4", output_file="output.mp3"):
        cmd = f"ffmpeg -i {input_file} -b:a 192K -vn {output_file}"
        print(cmd)
        os.system(cmd)

    def get_vedio_duration(self, file):
        return float(self.get_vedio_attribute(file)['format']['duration'])

    def convert_video(self, file):
        try:
            # target = os.path.join(self.output_folder, file) # 拼接文件名路径
            # try:
            #     if not os.path.exists(os.path.dirname(target)): # os.path.isdir(os.path.join(root, output)) os.path.join(root, output)
            #         os.makedirs(os.path.dirname(target))
            # except Exception as e:
            #     print('have error when create subfolder:',e)

            target = os.path.join(self.temp_folder, file)  # 拼接文件名路径
            try:
                # print('==================', os.path.exists(os.path.dirname(target)))
                # os.path.isdir(os.path.join(root, output)) os.path.join(root, output)
                if not os.path.exists(os.path.dirname(target)):
                    os.makedirs(os.path.dirname(target))
            except Exception as e:
                print('have error when create temp folder:', e)
                traceback.print_exc()
            video = Vedio(file)

            if self.remove_watermark:
                total_seconds = video.duration
                video.ended_time = total_seconds - 3.6
                video.temp_file = os.path.join(self.temp_folder, file)

            self.max_x_list.append(video.width)
            self.max_y_list.append(video.height)
            self.video_list.append(video)  # 将加载完后的视频加入列表

        except Exception as e:
            print('have error:', e)
            traceback.print_exc()
        finally:
            print(file, 'done')

    def combine_videos(self):
        self.max_x = max(self.max_x_list)
        self.max_y = max(self.max_y_list)

        executor = ThreadPoolExecutor(max_workers=self.max_workers)
        all_task = [executor.submit(self.cut_video, (video))
                    for video in self.video_list]
        wait(all_task, return_when=ALL_COMPLETED)
        current_folder = pathlib.Path(__file__).parent.resolve()
        with open('./list.txt', 'w', encoding='utf-8') as fin:
            fin.writelines([f"file '{os.path.join(current_folder, video.temp_file)}'\r\n".replace(
                '\\', '/') for video in self.video_list])

        self.call_combine_video()
        self.transfer_video_to_audio()

        shutil.rmtree(self.temp_folder) if os.path.exists(
            self.temp_folder) else None

    def preprocess_videos(self):
        files = glob.glob(f'{self.input_folder}/*.mp4', recursive=True)
        executor = ThreadPoolExecutor(max_workers=self.max_workers)
        all_task = [executor.submit(self.convert_video, (file))
                    for file in files]
        wait(all_task, return_when=ALL_COMPLETED)


if __name__ == "__main__":
    start_time = datetime.now()

    files = glob.glob('**/*.mp4', recursive=True)
    if not files:
        print('no files found')
        exit(0)
    dirs = list(set(['.' if os.path.dirname(file) ==
                '' else os.path.dirname(file) for file in files]))
    for dir in dirs:
        tiktok_util = TiktokUtil(input_folder=dir)
        tiktok_util.preprocess_videos()
        tiktok_util.combine_videos()

    ended_time = datetime.now()
    print(f'time cost: {ended_time - start_time}')

```

## 参考

- [ffmpeg合并多个MP4视频](https://blog.csdn.net/first_shun/article/details/108502532)
- [ffmpeg 命令详解](https://www.cnblogs.com/duwei/p/ffmpeg_commands.html)
