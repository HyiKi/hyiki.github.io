---
title: Python拼接照片墙
date:  2019-12-19 16:20:36 +0800
category: Reference
tags: Python
excerpt: 互动-利用Python的PIL库拼接照片墙
---

### PIL 支持的图像文件格式

BMP BUFR (identify only) CUR (read only) DCX (read only) EPS (write-only) FITS (identify only) FLI, FLC (read only) FPX (read only) GBR (read only) GD (read only) GIF GRIB (identify only) HDF5 (identify only) ICO (read only) IM IMT (read only) IPTC/NAA (read only) **JPEG** MCIDAS (read only) MIC (read only) MPEG (identify only) MSP PALM (write only) PCD (read only) PCX PDF (write only) PIXAR (read only) **PNG** PPM PSD (read only) SGI (read only) SPIDER TGA (read only) TIFF WAL (read only) WMF (identify only) XBM XPM (read only) XV Thumbnails

### python基本实现代码

```
from math import sqrt

import PIL.Image as Image
import os

IMAGES_PATH = './images/'  # 图片集地址
IMAGES_FORMAT = ['.jpg', '.jpeg', '.png']  # 图片格式
IMAGE_SIZE = 256  # 每张小图片的大小
IMAGE_ROW = 10  # 图片间隔，也就是合并成一张图后，一共有几行
IMAGE_COLUMN = 10  # 图片间隔，也就是合并成一张图后，一共有几列
IMAGE_SAVE_PATH = './test.png'  # 图片转换后的地址

# 获取图片集地址下的所有图片名称
image_names = [name for name in os.listdir(IMAGES_PATH) for item in IMAGES_FORMAT if
               os.path.splitext(name)[1] == item]

#图片数量
img_len = len(image_names)

# 简单的对于参数的设定和实际图片集的大小进行数量判断
# if img_len != IMAGE_ROW * IMAGE_COLUMN:
#     print("合成图片的参数和要求的数量不能匹配！")

#根据图片数量照片墙布局调整
if img_len <= 100 :
    IMAGE_ROW = IMAGE_COLUMN = int(sqrt(img_len))

# 定义图像拼接函数
def image_compose():
    to_image = Image.new('RGBA', (IMAGE_COLUMN * IMAGE_SIZE, IMAGE_ROW * IMAGE_SIZE))  # 创建一个新图
    # 循环遍历，把每张图片按顺序粘贴到对应位置上
    for y in range(1, IMAGE_ROW + 1):
        for x in range(1, IMAGE_COLUMN + 1):
            from_image = Image.open(IMAGES_PATH + image_names[IMAGE_COLUMN * (y - 1) + x - 1]).resize(
                (IMAGE_SIZE, IMAGE_SIZE), Image.ANTIALIAS)
            to_image.paste(from_image, ((x - 1) * IMAGE_SIZE, (y - 1) * IMAGE_SIZE))
    to_image.save(IMAGE_SAVE_PATH)  # 保存新图


image_compose()  # 调用函数
```