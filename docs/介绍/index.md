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

### 自动内存管理

`OpenCV自动处理所有内存`

首先，像`std::vector`,[`cv::Mat`](https://docs.opencv.org/5.x/d3/d63/classcv_1_1Mat.html)等一些数据结构在被具备析构函数的函数或方法使用时，可以在需要时释放底层内存缓冲区。这意味着析构函数并不总是像 Mat 那样释放缓冲区。他们考虑了可能的数据共享。析构函数减少了缓冲矩阵计数器的引用次数。当且仅当引用计数器为零时，即没有其他结构引用同一缓冲区时，才会释放缓冲区。换句话说，当复制 Mat 实例时，并没有真正复制任何实际数据。相反，引用计数器会增加以记住相同数据的另一个所有者。

（如果不想共享缓冲区的Mat数据）还有 [`cv::Mat::clone` ](https://docs.opencv.org/5.x/d3/d63/classcv_1_1Mat.html#a03d2a2570d06dcae378f788725789aa4)方法可创建矩阵数据的完整副本。请参阅下面的示例：
{% highlight C++ linenos %}
// 创建一个 8Mb 的矩阵
Mat A(1000, 1000, CV_64F);
 
// 为同一个矩阵创建另一个指针B； 
// 这是一个即时操作，无论矩阵大小如何。
Mat B = A;
// 为 矩阵A 的第三行创建另一个指针C；也没有复制任何数据
// 此时AB共享内存，C是AB的第三行
Mat C = B.row(3);
// 为B也是A创建完整副本D
Mat D = B.clone();
// 将B的第5行复制到C，即将A的第5行复制到 A 的第 3 行。
B.row(5).copyTo(C);
// 现在让A和D共享数据；之后的修改版本
// A 仍然被 B 和 C 引用。
A = D;
// 现在使 B 成为一个空矩阵（不引用任何内存缓冲区）
// 但是A的修改版本仍然会被C引用，
// 尽管 C 只是原始 A 的一行,所以A的内存区不会被释放，而仅仅是B为空了。
B.release();
 
// 最后，对 C 进行完整复制。（C不再引用A了）
// 因此，矩阵A将被释放，因为它没有被任何人引用
C = C.clone();
{% endhighlight %}

由此可见Mat等基本结构的使用很简单。但是，在创建没有考虑自动内存管理的高级类甚至用户数据类型的情况下阁下又该如何应对呢？为此，OpenCV 提供了 `cv::Ptr` 模板类，类似于 `C++11` 中的 `std::shared_ptr`。因此，不要使用普通指针：

{% highlight C++ linenos %}
T* ptr = new T(...);
{% endhighlight %}

你可以使用：

{% highlight C++ linenos %}
Ptr<T> ptr(new T(...));
{% endhighlight %}

或：

{% highlight C++ linenos %}
Ptr<T> ptr=makePtr<T>(...);
{% endhighlight %}

`Ptr<T>` 封装了一个指向 T 实例的指针以及与该指针关联的引用计数器。有关详细信息，请参阅 [`cv::Ptr`](https://docs.opencv.org/5.x/dc/d84/group__core__basic.html#ga6395ca871a678020c4a31fadf7e8cc63) 描述。


### 输出数据自动分配

大多数时候，OpenCV 会自动释放内存，并自动为输出函数参数分配内存。因此，如果函数具有一个或多个输入数组（cv::Mat 实例）和一些输出数组，则输出数组会自动分配或重新分配内存。输出数组的大小和类型由输入数组的大小和类型确定。如果需要，这些函数会采用额外的参数来帮助确定输出数组的属性。

例如：

{% highlight C++ linenos %}
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
 
using namespace cv;
 
int main(int, char**)
{
    VideoCapture cap(0);
    if(!cap.isOpened()) return -1;
 
    Mat frame, edges;
    namedWindow("edges", WINDOW_AUTOSIZE);
    for(;;)
    {
        cap >> frame;
        cvtColor(frame, edges, COLOR_BGR2GRAY);
        GaussianBlur(edges, edges, Size(7,7), 1.5, 1.5);
        Canny(edges, edges, 0, 30, 3);
        imshow("edges", edges);
        if(waitKey(30) >= 0) break;
    }
    return 0;
}
{% endhighlight %}

由于视频捕获模块已知视频帧分辨率和位深度，因此帧数组由 >> 运算符自动分配。数组边缘由 `cvtColor` 函数自动分配。它具有与输入数组相同的大小和位深度。通道数为1，因为传递了颜色转换代码[`cv::COLOR_BGR2GRAY`](https://docs.opencv.org/5.x/d8/d01/group__imgproc__color__conversions.html#gga4e0972be5de079fed4e3a10e24ef5ef0a353a4b8db9040165db4dacb5bcefb6ea)，表示颜色到灰度的转换。
请注意，在第一次执行循环体期间，帧和边缘仅分配一次，因为所有接下来的视频帧都具有相同的分辨率。如果您以某种方式更改视频分辨率，阵列会自动重新分配。

该技术的关键组件是 [`cv::Mat::create`](https://docs.opencv.org/5.x/d3/d63/classcv_1_1Mat.html#a55ced2c8d844d683ea9a725c60037ad0) 方法。它采用按需分配数组大小和类型。如果数组已经指定大小和类型，则该方法不执行任何操作。否则，它释放先前分配的数据（如果有）（这部分涉及引用计数器并将其与零进行比较），然后分配所需大小的新缓冲区。大多数函数为每个输出数组调用 `cv::Mat::create` 方法，从而实现自动输出数据分配。

该方案的一些值得注意的例外是 [`cv::mixChannels`](https://docs.opencv.org/5.x/d2/de8/group__core__array.html#ga51d768c270a1cdd3497255017c4504be)、[`cv::RNG::fill`](https://docs.opencv.org/5.x/d1/dd6/classcv_1_1RNG.html#ad26f2b09d9868cf108e84c9814aa682d) 以及一些其他函数和方法。它们无法分配输出数组，因此您必须提前执行此操作。

### 饱和度算术公式

作为一个计算机视觉库，OpenCV 需要处理大量图像像素，这些像素通常以紧凑的、每通道 8 或 16 位的形式进行编码，因此值范围有限。此外，对图像的某些操作，如色彩空间转换、亮度/对比度调整、锐化、复杂插值（bi-cubic,、Lanczos）可能会产生超出可用范围的值。如果仅存储结果的最低 8 (16) 位，则会导致视觉伪影，并可能影响进一步的图像分析。为了解决这个问题，使用了所谓的饱和算法。例如，要将运算结果 r 存储到 8 位图像，您可以找到 0..255 范围内最接近的值：


<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>I</mi>
  <mo stretchy="false">(</mo>
  <mi>x</mi>
  <mo>,</mo>
  <mi>y</mi>
  <mo stretchy="false">)</mo>
  <mo>=</mo>
  <mo data-mjx-texclass="OP" movablelimits="true">min</mo>
  <mo stretchy="false">(</mo>
  <mo data-mjx-texclass="OP" movablelimits="true">max</mo>
  <mo stretchy="false">(</mo>
  <mrow data-mjx-texclass="ORD">
    <mtext>round</mtext>
  </mrow>
  <mo stretchy="false">(</mo>
  <mi>r</mi>
  <mo stretchy="false">)</mo>
  <mo>,</mo>
  <mn>0</mn>
  <mo stretchy="false">)</mo>
  <mo>,</mo>
  <mn>255</mn>
  <mo stretchy="false">)</mo>
</math>

类似的规则适用于 8 位有符号、16 位有符号和无符号类型。这种语义在库中随处可见。在 C++ 代码中，它是使用类似于标准 C++ 转换操作的 [`cv::saturate_cast<>`](https://docs.opencv.org/5.x/db/de0/group__core__utils.html#gab93126370b85fda2c8bfaf8c811faeaf)函数来完成的。上面提供的公式的实现如下：

{% highlight C++ linenos %}
I.at<uchar>(y, x) = saturate_cast<uchar>(r);
{% endhighlight %}

其中 `cv::uchar` 是 OpenCV 8 位无符号整数类型。优化后的SIMD代码中使用了paddusb、packuswb等SSE2指令。它们有助于实现与 C++ 代码中完全相同的行为。

{: .note}
注意：当结果是 32 位整数时，饱和度公式不适用。

### 固定像素类型 - 模板的使用限制

模板是 C++ 的一项重要功能，可以实现非常强大、高效且安全的数据结构和算法。然而，模板的广泛使用可能会显着增加编译时间和代码大小。此外，当单独使用模板时，很难将接口和实现分开。这对于基本算法来说可能没问题，但对于计算机视觉库来说就不好了，因为单个算法可能跨越数千行代码。因此，也为了简化其他语言的绑定开发，例如根本没有模板或模板功能有限的 Python、Java、Matlab，当前的 OpenCV 实现基于多态性和模板上的运行时调度。在那些运行时调度太慢（如像素访问运算符）、不可能的地方（通用 cv::Ptr<> 实现），或者只是非常不方便（cv::saturate_cast<>()）当前的实现引入了小的模板类、方法和函数。在当前 OpenCV 版本的任何其他地方，模板的使用都是有限的。

因此，库可以操作的原始数据类型是有限的。也就是说，数组元素应该包含以下类型之一：

+ 8-bit unsigned integer (uchar)
+ 8-bit signed integer (schar)
+ 16-bit unsigned integer (ushort)
+ 16-bit signed integer (short)
+ 32-bit signed integer (int)
+ 32-bit floating-point number (float)
+ 64-bit floating-point number (double)
+ 由多个元素组成的元组，其中所有元素都具有相同的类型（上述类型之一）。其元素为此类元组的数组称为多通道数组，与元素为标量值的单通道数组相反。最大可能的通道数由 CV_CN_MAX 常量定义，当前设置为 512。

对于这些基本类型，应用以下枚举：

{% highlight C++ linenos %}
enum { CV_8U=0, CV_8S=1, CV_16U=2, CV_16S=3, CV_32S=4, CV_32F=5, CV_64F=6 };
{% endhighlight %}

可以使用以下选项指定多通道（n 通道）类型：

+ [CV_8UC1](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#ga81df635441b21f532fdace401e04f588) ... [CV_64FC4](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#ga44a3c8b22264a8a3e392d8245b0b1d37) 常量（适用于 1 到 4 的通道数） 当通道数大于 4 
+ 或编译时未知时，使用 [CV_8UC(n)](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#ga78c5506f62d99edd7e83aba259250394) ... [CV_64FC(n)](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#ga4213eb262159eb6da4edf8c9255e8244) 或 [CV_MAKETYPE(CV_8U, n)](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#gab2ebca36079fd923483abee99d7ff40d) ... [CV_MAKETYPE(CV_64F, n)](https://docs.opencv.org/5.x/d1/d1b/group__core__hal__interface.html#gab2ebca36079fd923483abee99d7ff40d) 宏。


{: .note}
#CV_32FC1 == #CV_32F、#CV_32FC2 == #CV_32FC(2) == #CV_MAKETYPE(CV_32F, 2) 和 #CV_MAKETYPE(深度, n) == ((深度&7) + ((n-1)<<3 ）。这意味着常量类型由深度（取最低 3 位）和通道数减 1（取接下来的 log2(CV_CN_MAX) 位）构成。

例如：

{% highlight C++ linenos %}
Mat mtx(3, 3, CV_32F); // make a 3x3 floating-point matrix
Mat cmtx(10, 1, CV_64FC2); // make a 10x1 2-channel floating-point
                           // matrix (10-element complex vector)
Mat img(Size(1920, 1080), CV_8UC3); // make a 3-channel (color) image
                                    // of 1920 columns and 1080 rows.
Mat grayscale(img.size(), CV_MAKETYPE(img.depth(), 1)); // make a 1-channel image of
                                                        // the same size and same
                                                        // channel type as img


{% endhighlight %}

使用 OpenCV 无法构建或处理具有更复杂元素的数组。此外，每个函数或方法只能处理所有可能数组类型的子集。通常，算法越复杂，支持的格式子集越小。请参阅以下此类限制的典型示例：

+ 人脸检测算法仅适用于 8 位灰度或彩色图像。
+ 线性代数函数和大多数机器学习算法仅适用于浮点数组。
+ 基本函数，例如 [cv::add](https://docs.opencv.org/5.x/d2/de8/group__core__array.html#ga10ac1bfb180e2cfda1701d06c24fdbd6)，支持所有类型。
+ 颜色空间转换函数支持 8 位无符号、16 位无符号和 32 位浮点类型。

每个功能支持的类型子集是根据实际需要定义的，并且将来可以根据用户请求进行扩展。

### 输入数组和输出数组

许多 OpenCV 函数处理密集的二维或多维数值数组。通常，此类函数以 cv::Mat 作为参数，但在某些情况下，使用 std::vector<> （例如，对于点集）或 cv::Matx<> （对于 3x3 单应矩阵等）会更方便。为了避免 API 中出现许多重复，引入了特殊的“代理”类。基本“代理”类是 [cv::InputArray](https://docs.opencv.org/5.x/dc/d84/group__core__basic.html#ga353a9de602fe76c709e12074a6f362ba)。它用于在函数输入上传递只读数组。从InputArray 派生的类[cv::OutputArray](https://docs.opencv.org/5.x/dc/d84/group__core__basic.html#gaad17fda1d0f0d1ee069aebb1df2913c0) 用于指定函数的输出数组。通常情况下，你不应该关心那些中间类型（并且你不应该显式声明这些类型的变量） - 它都会自动工作。您可以假设您始终可以使用 cv::Mat、std::vector<>、cv::Matx<>、cv::Vec<> 或 cv::Scalar，而不是 InputArray/OutputArray，当函数具有可选的输入或输出数组，而您没有或不需要时，请传递 [cv::noArray()](https://docs.opencv.org/5.x/dc/d84/group__core__basic.html#gad9287b23bba2fed753b36ef561ae7346)。


### 错误处理

OpenCV 使用异常来表示严重错误。当输入数据格式正确且属于指定的取值范围，但由于某种原因算法无法成功时（例如优化算法没有收敛），它返回一个特殊的错误代码（通常只是一个布尔变量）。

异常可以是 [cv::Exception](https://docs.opencv.org/5.x/d1/dee/classcv_1_1Exception.html) 类或其派生类的实例。反过来，cv::Exception 是 std::Exception 的派生物。因此可以使用其他标准 C++ 库组件在代码中优雅地处理它。

通常使用 [#CV_Error(errcode, description)](https://docs.opencv.org/5.x/db/de0/group__core__utils.html#ga5b48c333c777666e076bd7052799f891) 宏或其类似 printf 的 [#CV_Error_(errcode, (printf-spec, printf-args))](https://docs.opencv.org/5.x/db/de0/group__core__utils.html#ga5b48c333c777666e076bd7052799f891) 变体抛出异常，或者使用 [CV_Assert(condition) ](https://docs.opencv.org/5.x/db/de0/group__core__utils.html#gaf62bcd90f70e275191ab95136d85906b)宏来检查条件并在不满足时引发异常。对于性能关键型代码，[CV_DbgAssert(condition)](https://docs.opencv.org/5.x/db/de0/group__core__utils.html#gafbcb487cba05bd288dbe18c433de4f6f) 仅保留在调试配置中。由于自动内存管理，如果突然发生错误，所有中间缓冲区都会自动释放。如果需要，您只需要添加一条 try 语句来捕获异常：

{% highlight C++ linenos %}
try
{
    ... // call OpenCV
}
catch (const cv::Exception& e)
{
    const char* err_msg = e.what();
    std::cout << "exception caught: " << err_msg << std::endl;
}
{% endhighlight %}

### 多线程和可重入性

当前的 OpenCV 实现是完全可重入的。也就是说，不同类实例的相同函数或相同方法可以从不同线程调用。此外，相同的 Mat 可以在不同的线程中使用，因为引用计数操作使用特定于体系结构的原子指令。


