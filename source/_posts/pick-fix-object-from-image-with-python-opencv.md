---
title: 使用Python OpenCV提取图片中的特定物体
date: 2018-06-14 09:41:12
tags: [opencv,python,图像物体识别]
---

## OpenCV

`OpenCV`是一个基于BSD许可（开源）发行的跨平台计算机视觉库，可以运行在Linux、Windows、Android和Mac OS操作系统上。它轻量级而且高效——由一系列`C`函数和少量`C++`类构成，同时提供了`Python`、`Ruby`、`MATLAB`等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。

## HSV颜色模型

`HSV（Hue, Saturation, Value）`是根据颜色的直观特性由`A. R. Smith`在1978年创建的一种颜色空间, 也称六角锥体模型（`Hexcone Model`）。、这个模型中颜色的参数分别是：色调（H），饱和度（S），亮度（V）。

目前在计算机视觉领域存在着较多类型的颜色空间（`color space`）。HSV是其中一种最为常见的颜色模型，它重新影射了`RGB`模型，从而能够视觉上比`RGB`模型更具有视觉直观性。

一般对颜色空间的图像进行有效处理都是在`HSV`空间进行的，`HSV`的取值范围如下：

```
H:  0 ~ 180

S:  0 ~ 255

V:  0 ~ 255
```

## 目标

![原图](images/opencv-sample-box.png "原图")

这是我们的原图，我们希望把图片中间的绿色区域“扣”出来。

## 代码示例

源码地址[image_cutter](https://github.com/wueason/image_cutter)

```python
#!/usr/bin/env python
import cv2
import numpy as np


def find_center_point(file, blue_green_red=[], target_range=(), DEBUG=False):
    result = False
    if not blue_green_red:
        return result

    # 偏移量
    thresh = 30
    hsv = cv2.cvtColor(np.uint8([[blue_green_red]]), cv2.COLOR_BGR2HSV)[0][0]
    lower = np.array([hsv[0] - thresh, hsv[1] - thresh, hsv[2] - thresh])
    upper = np.array([hsv[0] + thresh, hsv[1] + thresh, hsv[2] + thresh])

    # 载入图片
    img = cv2.imread(file)

    # 获取图片HSV颜色空间
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

    # 获取遮盖层
    mask = cv2.inRange(hsv, lower, upper)

    # 模糊处理
    blurred = cv2.blur(mask, (9, 9))

    # 二进制化
    ret,binary = cv2.threshold(blurred, 127, 255, cv2.THRESH_BINARY)

    # 填充大空隙
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (21, 7))
    closed = cv2.morphologyEx(binary, cv2.MORPH_CLOSE, kernel)

    # 填充小斑点
    erode = cv2.erode(closed, None, iterations=4)
    dilate = cv2.dilate(erode, None, iterations=4)

    # 查找轮廓
    _, contours, _ = cv2.findContours(
        dilate.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    i = 0
    centers = []
    for con in contours:
        # 轮廓转换为矩形
        rect = cv2.minAreaRect(con)
        if not (target_range and 
            rect[1][0] >= target_range[0] - 5 and
            rect[1][0] <= target_range[0] + 5 and
            rect[1][1] >= target_range[1] - 5 and
            rect[1][1] <= target_range[1] + 5):
            continue
        centers.append(rect)
        if DEBUG:
            # 矩形转换为box对象
            box=np.int0(cv2.boxPoints(rect))

            # 计算矩形的行列起始值
            y_right = max([box][0][0][1], [box][0][1][1],
                          [box][0][2][1], [box][0][3][1])
            y_left  = min([box][0][0][1], [box][0][1][1],
                          [box][0][2][1], [box][0][3][1])
            x_right = max([box][0][0][0], [box][0][1][0],
                          [box][0][2][0], [box][0][3][0])
            x_left  = min([box][0][0][0], [box][0][1][0],
                          [box][0][2][0], [box][0][3][0])

            if y_right - y_left > 0 and x_right - x_left > 0:
                i += 1
                # 裁剪目标矩形区域
                target = img[y_left:y_right, x_left:x_right]
                target_file = 'target_{}'.format(str(i))
                cv2.imwrite(target_file + '.png', target)
                cv2.imshow(target_file, target)


            print('rect: {}'.format(rect))
            print('y: {},{}'.format(y_left, y_right))
            print('x: {},{}'.format(x_left, x_right))

    if DEBUG:
        cv2.imshow('origin', img)
        cv2.waitKey(0)
        cv2.destroyAllWindows()
    return centers

if __name__ == '__main__':
    # 目标的 bgr 颜色值，请注意顺序
    # 左边的绿色盒子
    bgr = [40, 158, 31]

    # 右边的绿色盒子
    # bgr = [40, 158, 31]

    point = find_center_point('opencv-sample-box.png',
                                blue_green_red=bgr,
                                DEBUG=True)
    # 中心坐标
    # point: [((152.0, 152.0), (63.99999237060547, 61.99999237060547), -0.0)]
    print(point[0][0][0])
```

运行之后我们得到了我们的目标图区域：

![目标图](images/opencv-target.png "目标图")

一般来说，我们会选择一些比较纯净的颜色区块，从而比较容易控制噪点，提高准确率。
