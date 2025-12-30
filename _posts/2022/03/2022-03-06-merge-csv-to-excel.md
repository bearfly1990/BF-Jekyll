---
layout: post
title: Merge CSV to Excel
subtitle: Merge csv files in folders to excel
date: 2022-03-06
author: BF
thumbnail: /img/bf/python.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/bf/python.jpg
tags:
    - python
    - pandas
    - csv
    - excel
    - merge
---

### Overview

周五老板说有没有用python把csv合并到excel中的例子，用python还是比较方便的。

<!--more-->
这次主要用到的库：

```properties
et-xmlfile==1.1.0
numpy==1.22.2
openpyxl==3.0.9
pandas==1.4.1
python-dateutil==2.8.2
pytz==2021.3
six==1.16.0
XlsxWriter==3.0.3
```

### 配置文件

使用配置文件`config.ini`来管理，可以支持目录，分隔符，有没有带headerr，和输出文件的配置。

```ini
[config]
;folder store csvs
folder=./data-normal/
;separator
sep=,
;csv with header: 1, no header: 0
header=1
;output header0
output=./output.normal.xlsx

;folder=./data-noheader/
;sep=;
;header=0
;output=./output.noheader.xlsx

; folder=./data-sep-tab/
; sep=\t
; header=0
; output=./output.tab.xlsx
```

### 代码

下面是主要的代码:

```python
import configparser
import glob
import os

import pandas as pd

pd.io.formats.excel.ExcelFormatter.header_style = None

CONFIG_PATH = './config.ini'


class Config:
    def __init__(self):
        cf = configparser.ConfigParser()
        cf.read(CONFIG_PATH)

        self.folder = cf.get("config", "folder")
        self.sep = cf.get("config", "sep")
        self.header = True if int(cf.get("config", "header")) == 1 else False
        self.output = cf.get("config", "output")


def read_csvs(config):
    folder = config.folder
    csv_file_paths = glob.glob(f"{folder}/*.csv", recursive=True)
    if not csv_file_paths:
        print('No csv files found, please check the config!')
        exit()
    with pd.ExcelWriter(config.output, engine="openpyxl") as writer:
        for csv_file_path in csv_file_paths:
            print(f'reading {csv_file_path}')
            df = pd.read_csv(csv_file_path, sep=config.sep, engine='python', header=0 if config.header else None)

            filename = os.path.splitext(os.path.basename(csv_file_path))[0]

            df.to_excel(writer, sheet_name=filename, index=None, header=True if config.header else None)
    print(f'write to {config.output} done')


if __name__ == '__main__':
    config = Config()
    read_csvs(config)
```

其中有几个细节，pandas默认输出的excel里header是加粗并且带线的，如果想要去掉就需要配置一下

```python
pd.io.formats.excel.ExcelFormatter.header_style = None
```

默认的方法是不支持`\t`这样的分隔符来表示的，需要加上`engine="openpyxl"`。

```python
with pd.ExcelWriter(config.output, engine="openpyxl") as writer:
    pass
```

### Demo

下面是简单的演示

[Merge CSV to Excel演示地址](https://www.douyin.com/video/7071888650568518919)
<!-- <video id="video" controls="" preload="none" poster="/img/bf/python.jpg">
      <source id="mp4" src="/vedio/2022/03/06/merge-csv-to-excel.mp4" type="video/mp4">
      </video> -->

### 源码

[merge-csv-to-excel.zip](/document/merge-csv-to-excel.zip)

### 参考

[详解pandas的read_csv方法](https://www.cnblogs.com/traditional/p/12514914.html)
[Pands Input/output](https://pandas.pydata.org/pandas-docs/stable/reference/io.html)
