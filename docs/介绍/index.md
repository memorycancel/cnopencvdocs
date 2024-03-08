---
title: 介绍
layout: default
nav_order: 2
---

# 介绍

原文：[https://docs.opencv.org/5.x/d1/dfb/intro.html](https://docs.opencv.org/5.x/d1/dfb/intro.html)

OpenCV（开源计算机视觉库：https://opencv.org ）是包含成百上千计算机视觉算法的开源库。本文介绍OpenCV 2.x版本API，本质上是C++ API，它不同于基于C的OpenCV 1.x API（C API已弃用并在OpenCV 2.4版本发布后就停止测试了）。

OpenCV 具有模块化结构，这意味着该软件包包含多个共享或静态库。可以使用以下模块：

+ [Core functionality](/docs/主要模块/core/)（core）核心功能 - 定义基础数据结构的重要模块，包括密集多维数组（dense multi-dimensional array）`Mat`和被所有其他模块使用的基础函数。
+ [Image Processing](/docs/主要模块/imgproc/)（imgproc）图像处理 - 图像处理模块包含线性和非线性过滤，图像几何变换（调整大小、仿射和透视变形、基于通用格子的重新映射），颜色空间转换，直方图，等等。（图像的传统经典处理算法）
+ [Video Analysis](/docs/主要模块/video)（video）- 视频分析模块包含动作辨识，去除背景以及对象追踪算法。
+ [3d](/docs/主要模块/3d)（3d）- 基础的多视图几何算法，对象动作辨识以及3D元素重建。
+ [2d Features Framework](/docs/主要模块/features2d)（features2d）- 显著特征检测、显著特征描述、显著特征描述匹配。（传统图像检索算法）
+ [Object Detection](/docs/主要模块/objdetect)（objdetect）- 对象检测，检测物体对象或预定义类的实例（例如人脸、眼睛、鬼脸、人、车、等等）。
+ [Camera Calibration](/docs/主要模块/calib)（calib）- 相机校准，单相机或三维立体相机校准。
+ [Stereo Correspondence](/docs/主要模块/stereo)（stereo）- 三维立体定位。
+ [High-level GUI](/docs/主要模块/highgui)（highgui）- 易于使用并具备简单的UI功能的界面。
+ [Video I/O](/docs/主要模块/videoio)（videoio）- 易于使用的视频捕获和编解码器功能的界面。
+ ... ... 等一些其他的帮助模块，例如：FLANN、谷歌测试、Python接口等。

本文档的后续章节描述了每个模块的功能。但首先请确保熟悉库中API使用的一些常见概念。
PS：默认您拥有C++编码基础。

---

## API使用常见概念

### `cv`命名空间

所有的OpenCV类和函数都放在`cv`命名空间下。所以在您的代码使用本库功能时，使用`cv::`指定符或`using namespace cv;`指令，例如：

{% highlight C++ linenos %}
#include "opencv2/core.hpp"
//...
cv::Mat H=cv::findHomography(points1,points2,cv::RANSAC,5);
//...
{% endhighlight %}

或：
{% highlight C++ linenos %}

#include "opencv2/core.hpp"
using namespace cv;
Mat H=findHomography(points1,points2,RANSAC,5);
{% endhighlight %}

当前或未来的一些 OpenCV 功能命名可能与 STL 或其他库冲突。在这种情况下，请使用显式命名空间说明符来解决名称冲突：

{: .note}
PS： STL(Standard Template Library)，即标准模板库,是一个具有工业强度的，高效的C++程序库。它被容纳于C++标准程序库(C++ Standard Library)中，是ANSI/ISO C++标准中最新的也是极具革命性的一部分。

{% highlight C++ linenos %}
#include "opencv2/core.hpp"
Mat a(100, 100, CV_32F);
randu(a, Scalar::all(1), Scalar::all(std::rand()));
cv::log(a, a); //cv::log
a /= std::log(2.); //std::log
{% endhighlight %}