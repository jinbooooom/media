## [Opencv基础](https://www.cnblogs.com/Undo-self-blog/p/8423851.html)

### 图像的通道

- 1 通道为灰度图；
- 2  通道的图像是RGB555和RGB565。2通道图在程序处理中会用到，如傅里叶变换，可能会用到，一个通道为实数，一个通道为虚数，主要是编程方便。RGB555是16位的，2个字节，5+6+5，第一字节的前5位是R，后三位+第二字节是G，第二字节后5位是B，可见对原图像进行压缩了
- 3 通道为彩色图（RGB）；
- 4 通道为 RGBA ，是RGB加上一个A通道，也叫alpha通道，表示透明度，PNG图像是一种典型的4通道图像。alpha通道可以赋值0到1，或者0到255，表示透明到不透明

**大部分使用场景下，常使用的是1，3，4通道； 2通道不常见**

### CvType 类型常量组合规则

```
CV_[bite](U|S|F)C[channels]
```

- **bite** : 比特数，位数。 有 8bite，16bite，32bite，64bite,对应在 Mat 中，每个像素的所占的空间大小，8位即 CV_8

- **U|S|F** ：

  - U : unsigned int , 无符号整形
  - S : signed int , 有符号整形
  - F : float , 单精度浮点型,float类型本身即有符号

  > 这里的有符号、无符号是针对图像二进制编码来讲的。我在写的过程中大多数情况下都是使用的无符号，即 CV_8U ,CV_16U，当有计算时可能会介入有符号（存在负数），没学过 C++，对底层也一知半解，望高手解答。

- **C[channels]**：图像的通道数

通过上边的解释，我想您已经明白了个大概，比如 CV_8UC3 即 8位无符号的3通道（RGB 彩色）图像

[OpenCV 3.4  读懂 CvType](https://my.oschina.net/u/3767256/blog/1794173)

### Opencv 为什么用 BGR 而不是 RGB

RGB通道和BGR通道的区别是顺序不一样，其它没有什么区别。而opencv选择的是BGR，因为OpenCV的早期开发当中BGR颜色格式在相机制造商和软件提供商中很受欢迎。

（早期BGR也比较流行，opencv一开始选择了BGR，到后来即使RGB成为主流）

### RGB转换成BGR

```Python
# BGR to RGB
# OpenCV image to Matplotlib
rgb = bgr[...,::-1]
# RGB to BGR
# Matplotlib image to OpenCV
bgr = rgb[...,::-1]
# RGB to GBR   0->2 1->0 2->1
gbr = rgb[...,[2,0,1]]
# 其实 opencv 里有自带的转换函数，无论python还是C++
# cv::cvtColor(cv_img, cv_img, cv::COLOR_RGB2BGR);
```

### 图像 ROI

```Python
import cv2
import numpy as np
img=cv2.imread('messi5.jpg')
ball=img[280:340,330:390]
img[273:333,100:160]=ball
img=cv2.imshow('test', img)
cv2.waitKey(0)
```

### 拆分及合并图像通道

有时我们需要对 BGR 三个通道分别进行操作。这是你就需要把 BGR 拆分成单个通道。有时你需要把独立通道的图片合并成一个 BGR 图像。你可以这样做：

```
import cv2
import numpy as np
img=cv2.imread('/home/duan/workspace/opencv/images/roi.jpg')
b,g,r=cv2.split(img)
img=cv2.merge(b,g,r)
```

或者：

```
import cv2
import numpy as np
img=cv2.imread('/home/duan/workspace/opencv/images/roi.jpg')
b=img[:,:,0]
```

cv2.split() 是一个比较耗时的操作。只有真正需要时才用它，能用Numpy 索引就尽量用。

### **图像混合**

　　这其实也是加法，但是不同的是两幅图像的权重不同，这就会给人一种混合或者透明的感觉。图像混合的计算公式如下：
　　　　g (x) = (1 − α)f 0 (x) + αf 1 (x)

我们知道，图像的强度取值区间是[0, 255]，opencv的图像类型是uint8类型。而numpy的矩阵加法是一种模（mod）操作，即200+60=260 % 256 = 4，所以该亮的地方反而暗淡了。

opencv自带有图像加法操作运算函数：add(x, y)。add使用的是饱和操作，即200+60=260->255。所以使用opencv自带的add()函数效果会更好。

#### 图像去噪声

**均值滤波**是一种线性滤波，其核心思想是-领域平均法，均值滤波是用图像上一点的领域范围内所有像素的均值代替该点的值，经过均值计算后就可以达到去除突变噪声干扰的效果。而均值滤波的缺点是会造成图像模糊。

**形态学去噪：开闭运算去噪**

### 图像对比度、亮度值调整

g(i, j) = a * f(i, j) + b

a为增益，调整对比度；

b为偏置，调整亮度。

### 滤波

#### 三种线性滤波

两个信号的响应和它们各自的响应之和相等，换句话说，每个像素的输出值是一些输入像素的加权和。

- 方框滤波(boxFilter)
- 均值滤波（blur）：其内部实现就是调用了方框滤波。均值滤波是典型的线性滤波，主要方法是邻域平均法，即用一块图像区域的各个像素的均值来代替原图像中的各个像素值。
  - 缺陷：不能很好的保护细节。去噪的同时也破坏了图像的细节部分，从而使图像变得模糊。

- 高斯滤波（gaussianBlur）：是一种线性平滑滤波，可以消除高斯噪声。用核确定的邻域内像素的加权平均灰度值去替代模板中心像素点的值。

#### 两种非线性滤波

在很多情况下使用领域像素的非线性滤波会得到更好的效果。比如在噪声是散粒噪声而不是高斯噪声，即图像偶尔会出现很大的值的时候，用高斯滤波对图像模糊，噪声是不会被去除，只是被转换成更为柔和但依旧可见的颗粒。这个时候就需要中值滤波。

- 中值滤波：用像素点邻域灰度值的中值来代替该像素点的灰度值，该方法在去除脉冲噪声、椒盐噪声的同时又能很好的保留图像的边缘细节。
  - 优势：均值滤波中，由于噪声成分被放入平均计算中，所以输出受到了噪声的影响，但是在中值滤波中，由于噪声成分很难被选上，所以几乎不会影响到输出。
  - 劣势：中值滤波花费的时间是均值滤波的5倍以上，因为要排序。

- 双边滤波：结合图像的空间邻近度和像素值相似度的一种折中处理，同时考虑空域信息和灰度相似度，达到保边去噪的目的。

  用高斯滤波对高频细节保护的不明显，而双边滤波保留较多的高频信息，只能对低频信息进行较好地滤波。

  高频对应着图像变化剧烈的部分，也就是图像的边缘（轮廓），或者噪声以及细节部分；低频对应着图像的主要部分，也就是亮度或者灰度值变化缓慢的区域。

### 仿射变换

#### resize

resize的底层原理是[双线性插值](https://blog.csdn.net/qq_37577735/article/details/80041586)

### 轮廓

函数：cv2.findContours()，cv2.drawContours()

轮廓可以简单认为成将连续的点（连着边界）连在一起的曲线，具有相同、的颜色或者灰度。轮廓在形状分析和物体的检测和识别中很有用。

- 为了更加准确，要使用二值化图像。在寻找轮廓之前，要进行阈值化处理、或者 Canny 边界检测。
- 查找轮廓的函数会修改原始图像。如果你在找到轮廓之后还想使用原始图像的话，你应该将原始图像存储到其他变量中。
- 在 OpenCV 中，查找轮廓就像在黑色背景中找白色物体。你应该记住，要找的物体应该是白色而背景应该是黑色。

#### **轮廓面积**

```
area = cv2.contourArea(cnt)
```

#### **轮廓周长**

　　也被称为弧长。可以使用函数 cv2.arcLength() 计算得到。这个函数的第二参数可以用来指定对象的形状是闭合的（True），还是打开的（一条曲线）。

```
perimeter = cv2.arcLength(cnt,True)
```

#### **什么是层次结构**

　　通常我们使用函数 cv2.findContours 在图片中查找一个对象。有时对象可能位于不同的位置。还有些情况，一个形状在另外一个形状的内部。这种情况下我们称外部的形状为父，内部的形状为子。按照这种方式分类，一幅图像中的所有轮廓之间就建立父子关系。这样我们就可以确定一个轮廓与其他轮廓是怎样连接的，比如它是不是某个轮廓的子轮廓，或者是父轮廓。这种关系就成为组织结构。

### 形态学操作

#### **腐蚀** erode

（前景白背景黑）

卷积核沿着图像滑动，如果与卷积核对应的原图像的所有像素值都是 1，那么中心元素就保持原来的像素值，否则就变为零。

所以前景物体会变小，整幅图像的白色区域会减少。这对于去除白噪声很有用，也可以用来断开两个连在一块的物体。

#### 膨胀 dilate

与腐蚀相反，与卷积核对应的原图像的像素值中只要有一个是 1，中心元素的像素值就是 1。所以这个操作会增加图像中的白色区域（前景）。一般在去噪声时先用腐蚀再用膨胀。因为腐蚀在去掉白噪声的同时，也会使前景对象变小。所以我们再对他进行膨胀。这时噪声已经被去除了，不会再回来了，但是前景还在并会增加。膨胀也可以用来连接两个分开的物体。

#### **开运算**

　　先进行腐蚀再进行膨胀就叫做开运算。就像我们上面介绍的那样，它被用来去除噪声（白点），在纤细处分离物体。

#### **闭运算**

　　先膨胀再腐蚀。它经常被用来填充前景物体中的小洞（黑点）。

#### 形态学梯度

膨胀图与腐蚀图之差，可以突出前景的边缘轮廓（很像镂空的字体）

#### 顶帽（礼帽）

dst = src - open

原始图像与进行开运算之后得到的图像的差。

因为开运算是放大了裂缝或者局部低亮度的区域，因此从原图减去开运算，得到的效果图突出了比原图轮廓周围区域更明亮的区域。

（开运算先腐蚀去除了白点，再膨胀，原图再减去开运算后的图，就突出了白点）

#### **黑帽**

dst = close - src

进行闭运算之后得到的图像与原始图像之差。

突出了比原图轮廓周围区域更暗的区域。



[顶帽与黑帽](https://www.cnblogs.com/nickup/p/6013345.html)

### 阈值化

- 固定阈值
- 自适应阈值

### 边缘检测

### Canny 边缘检测

- **噪声去除**
  　　由于边缘检测很容易受到噪声影响，所以第一步是使用 5x5 的高斯平滑滤波器卷积去除噪声，这个前面我们已经学过了。

- **计算图像梯度**
  　　对平滑后的图像使用 Sobel 算子计算水平方向和竖直方向的一阶导数（图像梯度）（Gx 和 Gy）。根据得到的这两幅梯度图（Gx 和 Gy）找到边界的梯度和方向


- **非极大值抑制**
  　　在获得梯度的方向和大小之后，应该对整幅图像做一个扫描，去除那些非边界上的点。对每一个像素进行检查，看这个点的梯度是不是周围具有相同梯度方向的点中最大的。保留含有最大梯度的边界，就可以得到一个包含“窄边界”的二值图像。

- **滞后阈值**
  　　现在要确定那些边界才是真正的边界。这时我们需要设置两个阈值：minVal 和 
  maxVal。当图像的灰度梯度高于 maxVal 时被认为是真的边界，那些低于 minVal 
  的边界会被抛弃。如果介于两者之间的话，就要看这个点是否与某个被确定为真正的边界点相连，如果是就认为它也是边界点，如果不是就抛弃。在这一步一些小的噪声点也会被除去，因为我们假设边界都是一些长的线段。

### 图像金字塔

有两类图像金字塔：高斯金字塔和拉普拉斯金字塔。

高斯金字塔的顶部是通过将底部图像中的连续的行和列去除得到的。**顶部图像中的每个像素值等于下一层图像中 5 个像素的高斯加权平均值**。这样操作一次一个MxN 的图像就变成了一个 M/2xN/2 的图像。所以这幅图像的面积就变为原来图像面积的四分之一。这被称为 Octave。连续进行这样的操作我们就会得到一个分辨率不断下降的图像金字塔。

### 直方图

什么是直方图呢？通过直方图你可以对整幅图像的灰度分布有一个整体的了解。直方图的 x 轴是灰度值（0 到 255），y 轴是图片中具有同一个灰度值的点的数目。直方图其实就是对图像的另一种解释。通过直方图我们可以对图像的对比度，亮度，灰度分布等有一个直观的认识。（要记住，直方图是根据灰度图像绘制的，而不是彩色图像）。直方图的左边区域像是了暗一点的像素数量，右侧显示了亮一点的像素的数量。

#### 统计直方图

上面的直方图显示了每个灰度值对应的像素数。如果像素值为 0到 255，你就需要 256 个数来显示上面的直方图。但是，如果你不需要知道每一个像素值的像素点数目的，而只希望知道两个像素值之间的像素点数目怎么办呢？举例来说，我们想知道像素值在0 到 15 之间的像素点的数目，接着是 16 到 31,....，240 到 255。我们只需要 16 个值来绘制直方图。

OpenCVTutorials on histograms中例子所演示的内容。那到底怎么做呢？你只需要把原来的 256 个值等分成 16 小组，取每组的总和。而这里的每一个小组就被成为 BIN。第一个例子中有 256 个 BIN，第二个例子中有 16 个 BIN。在 OpenCV 的文档中用 histSize 表示 BINS

#### 直方图均衡化

如果一张图像的背景或者前景太亮或者太暗，那么大多数像素点的像素值就都集中在一个像素值范围之内，导致整张图片的对比度很低，这个时候就可以用图像直方图对对比度进行调整。一副高质量的图像的像素值分布应该很广泛，那么它的直方图就很平均，而一个太亮或者太暗的图，它的直方图是在某一个局部集中，直方图均衡就是把它的直方图做一个横向拉伸。

（如果一副图像中的大多数像素点的像素值都集中在一个像素值范围之内，例如，如果一幅图片整体很亮，那所有的像素值应该都会很高。但是一副高质量的图像的像素值分布应该很广泛。所以你应该把它的直方图做一个横向拉伸（如下图），这就是直方图均衡化要做的事情。通常情况下这种操作会改善图像的对比度。）

（另一个重要的特点是，即使我们的输入图片是一个比较暗的图片（不像上边我们用到到的整体都很亮的图片），在经过直方图均衡化之后也能得到相同的结果。因此，直方图均衡化经常用来使所有的图片具有相同的亮度条件的参考工具。这在很多情况下都很有用。例如，脸部识别，在训练分类器前，训练集的所有图片都要先进行直方图均衡化从而使它们达到相同的亮度条件。）

#### 自适应的直方图均衡化

有些图只是局部太暗或者太亮，对比度较低，而在全局上是比较均衡的，这个时候直接用（全局）直方图均衡化就不太合适，应该用自适应的直方图均衡化。

这种情况下，整幅图像会被分成很多小块，这些小块被称为“tiles”（在 OpenCV 中 tiles 的大小默认是 8x8），然后再对每一个小块分别进行直方图均衡化（跟前面类似）。

### 分水岭算法图像分割

任何一副灰度图像都可以被看成拓扑平面，灰度值高的区域可以被看成是山峰，灰度值低的区域可以被看成是山谷。我们向每一个山谷中灌不同颜色的水。随着水位升高，不同山谷的水就会相遇汇合，为了防止不同山谷的水汇合，我们需要在水汇合的地方构建起堤坝。不停的灌水，不停的构建堤坝知道所有的山峰都被水淹没。我们构建好的堤坝就是对图像的分割。这就是分水岭算法的背后哲理。

但是这种方法通常都会得到过度分割的结果，这是由噪声或者图像中其他不规律的因素造成的。为了减少这种影响，OpenCV 采用了基于掩模的分水岭算法，在这种算法中我们要设置那些山谷点会汇合，那些不会。这是一种交互式的图像分割。我们要做的就是给我们已知的对象打上不同的标签。如果某个区域肯定是前景或对象，就使用某个颜色（或灰度值）标签标记它。如果某个区域肯定不是对象而是背景就使用另外一个颜色标签标记。而剩下的不能确定是前景还是背景的区域就用0 标记。这就是我们的标签。然后实施分水岭算法。每一次灌水，我们的标签就会被更新，当两个不同颜色的标签相遇时就构建堤坝，直到将所有山峰淹没，最后我们得到的边界对象（堤坝）的值为-1

### 凸包



### 特征

#### 角点检测

对于角点来说，角点特征唯一，比较好检测

### 图像处理笔试面试题目

[图像连通域分析](https://blog.csdn.net/u011028345/article/details/80511822)

- https://zhuanlan.zhihu.com/p/31951244

