---
title: 图像处理pillow库
tags:
  - python
  - pillow
comments: true
date: 2021-06-30 12:19:24
categories: Python库
index_img:
banner_img:
typora-root-url: 图像处理pillow库
---

## 安装

Install Pillow with **pip**:

```shell
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade Pillow
```

## 教程

### 使用图像类

导入图像类，使用`open()`函数打开图片。

```python
from PIL import Image

im = Image.open(r"D:/images/my.jpg")
print(im)
# <PIL.PngImagePlugin.PngImageFile image mode=RGB size=3300x3300 at 0x199B5C860D0>
```

`Image.open()`会返回一个`Image`对象。

- `format` : 识别图像的源格式，如果该文件不是从文件中读取的，则被置为 None 值。
- `size` : 返回的一个元组，有两个元素，其值为象素意义上的宽和高。
- `mode` : `RGB（true color image）`，此外还有，`L（luminance）`，`CMTK（pre-press image）`。

```python
print(im.format, im.size, im.mode)
# PNG (3300, 3300) RGB
```

显示加载的图像，使用`image.show()`方法，回调用系统的默认工具打开图像，如果没有图像工具会提示报错。

```python
im.show()
```

### 读写图像

`Image`库支持几乎所有的图像格式，直接使用`open`函数打开文件即可，程序会自动探测文件的格式。

要保存图像到文件，使用`Image`类中的`save()`方法。

#### 转图像到JPG

```python
import os
import sys
from PIL import Image

for infile in sys.argv[1:]:
    f, e = os.path.splitext(infile)
    outfile = f + ".jpg"
    if infile != outfile:
        try:
            with Image.open(infile) as im:
                im.save(outfile)
        except OSError:
            print("cannot convert", infile)
```

上面的代码实现了把任意格式的图像文件转成`JPG`格式，并保存到文件所在的同一目录下。

#### 创建缩约图

```python
import os, sys
from PIL import Image

size = (128, 128)

for infile in sys.argv[1:]:
    outfile = os.path.splitext(infile)[0] + ".thumbnail"
    if infile != outfile:
        try:
            with Image.open(infile) as im:
                im.thumbnail(size)
                im.save(outfile, "JPEG")
        except OSError:
            print("cannot create thumbnail for", infile)
```

这里面的`size`元组定义了图像的大小。

### 切割图像

```
from PIL import Image

im = Image.open("D:/images/default.png")
im.show()
box = (100, 100, 400, 400)
region = im.crop(box)
region.show()
```

区域由 4 个坐标定义，坐标位于（左、上、右、下）。Python 成像库使用左上角有 （0， 0） 的坐标系统。另请注意，坐标是指像素之间的位置，因此上述示例中的区域正好为 `300x300` 像素。‎

#### 调整图像大小

```python
from PIL import Image
# 载入图像
image = Image.open("D:/images/default.png")
# 调整大小
new_image = image.resize((400, 400))

print("原图像的大小:", image.size)
print("新图像的大小:", new_image.size)

# 原图像的大小: (4096, 2305)
# 新图像的大小: (400, 400)
```

`image.resize`接收一个元组。

## 图像滤波

导入图片和库文件。

```python
from PIL import Image, ImageFilter
im = Image.open(r"D:/images/my.jpg")
```

#### BLUR：模糊滤波

```python
# 模糊滤波
bluF = im.filter(ImageFilter.BLUR)
bluF.show()
```

#### CONTOUR：轮廓滤波

```python
# 轮廓滤波
conF = im.filter(ImageFilter.CONTOUR)
conF.show()
```

![image-20210630172021291](/image-20210630172021291.png)

#### DETAIL：细节滤波

```python
# 细节滤波
detF = im.filter(ImageFilter.DETAIL)
detF.show()
```

#### EDGE_ENHANCE：边界增强滤波

```python
# 边界增强滤波
eeF = im.filter(ImageFilter.EDGE_ENHANCE)
eeF.show()
```

#### EMBOSS：浮雕滤波

```python
# 浮雕滤波
embF = im.filter(ImageFilter.EMBOSS)
embF.show()
```

![image-20210630172930687](/image-20210630172930687.png)

#### FIND_EDGES：寻找边界滤波（找寻图像的边界信息）

```python
# 寻找边界滤波（找寻图像的边界信息）
fdeF = im.filter(ImageFilter.FIND_EDGES)
fdeF.show()
```

![image-20210630173055303](/image-20210630173055303.png)

#### SMOOTH：平滑滤波

```python
# 平滑滤波
smoF = im.filter(ImageFilter.SMOOTH)
smoF.show()
```

#### SMOOTH_MORE：深度平滑滤波

```python
# 深度平滑滤波
smomF = im.filter(ImageFilter.SMOOTH_MORE)
smomF.show()
```

#### SHARPEN：锐化滤波

```python
# 锐化滤波
shaF = im.filter(ImageFilter.SHARPEN)
shaF.show()
```

#### GaussianBlur：高斯模糊

```python
# 高斯模糊
gbF = im.filter(ImageFilter.GaussianBlur(radius=10))
gbF.show()
```

![image-20210630173451367](/image-20210630173451367.png)









[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


