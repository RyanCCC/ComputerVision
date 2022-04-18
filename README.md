# 计算机视觉

本项目基于C++实现计算机视觉中的算法。包括传统的计算机视觉算法以及基于深度学习的算法。主要开发工具是visual studio 2019，后续会学习相关的CMake操作。另外在Python文件夹下有相关的opencv python代码，并且设置对应的readme介绍相对应的算法。

## 环境配置

### OpenCV

关于OpenCV的配置在网上有很多，在这里只做简单的介绍，详情请具体搜索相关的博客或者官网资料。

1. 从官网下载对应的opencv版本
2. 双击opencv，提取文件到预设定的文件夹。然后配置相关的环境变量，在控制台上输入```opencv_version```，提示opencv版本说明配置成功。
![image](https://user-images.githubusercontent.com/27406337/145359576-1e9b1107-a4f9-497d-9f2e-626ece48d468.png)
![image](https://user-images.githubusercontent.com/27406337/145359808-aa0529e9-89db-46a8-b899-616661b07fb0.png)
3. 在配置的opencv环境之后，需要配置工程的opencv环境。对你的Debug x64编译器配置opencv的头文件include和lib即可。
4. 注意一点：假设运行之后提示没有相关的dll，只需要把安装文件下的```opencv_world*.dll```等dll文件放到系统的可执行文件上即可。
5. 配置完成后即可继续遨游C++的知识海洋了


## 计算机视觉算法

### 去雾算法

去雾算法主要参考的博文以及论文分别如下：
1. 博文：https://bbs.huaweicloud.com/blogs/302935

2. 论文:《Single Image Haze Removal Using Dark Channel Prior》

- 大气散射模型

$$I(x)=J(x)t(x)+A(1-t(x))$$ 

其中$x$表示图像的空间坐标，$I(x)$表示有雾图像（待去雾图像），$I(x)$表示有雾图像（待去雾图像），A代表全球大气光值，$t(x)$代表投射值

- 暗通道定义

在绝大多数非天空的局域区域中，某些像素总会至少有一个颜色通道的值很低。对于一副图像$J(x)$，其暗通道的数学定义表示如下：

$$J^{dark}(X) = \min_{y \in \Omega(x)}(\min_{c\in\{r, g, b\}} J^{c}(y))$$

其中$\Omega(x)$表示以$x$为中心的局部区域，上标$c$表示RGB三个通道。首先求出每个像素RGB分量中的最小值，存入一副与原始图像大小相同的灰度图中，然后对这幅灰度图进行最小值滤波，滤波的半径由窗口大小决定。

- 暗通道先验理论

暗通道先验理论指出：对于非天空区域的无雾图像$J(x)$的暗通道趋向于0，即$$J^{dark}\to 0$$

实际生活中造成暗原色中低通道值主要有三个因素：
1. 汽车、建筑物和城市中玻璃窗户的投影，或者是输液、树与岩石等自然景观的投影
2. 色彩鲜艳的物体或表面，在RGB的三个通道中有些通道的值很低
3. 颜色较暗的物体或者表面，例如灰暗色的树干和石头

- 公式推导过程

根据大气散射模型，将第一个公式变形为下式：

$$\frac{I^{c}(X)}{A^{c}} = t(x)\frac{J^{c}(X)}{A^{c}} + 1 - t(x)$$

假设每一个窗口的透射率$t(x)$为常数，记为$\tilde{t}(x)$，并且A值已给定，对式子两边同时进行两次最小值运算，可得：

$$\min_{y\in \Omega(x)}(min_{c}\frac{I^{c}(y)}{A^{c}}) = \tilde{t}(x)\min_{y\in\Omega(x)}(\min_{c}\frac{J^{c}(y)}{A^{c}})+1-\tilde{t}(x)$$

其中，$J(x)$是要求的无雾图像，根据前述的暗通道先验理论可知：

$$J^{dark}(x) = \min_{y\in \Omega(x)}(\min_{c}J^{c}(y)) = 0$$

由此可以推出：

$$\min_{y\in \Omega(x)}(\min_{c}\frac{J^{c}(y)}{A^{c}})=0$$

因此可以得到透射率$\tilde{t}(x)$的预估值：

$$\tilde{t}(x) = 1- \min_{y\in \Omega(x)}(\min_{c}\frac{I^{c}(y)}{A^{c}})$$


在去雾的同时要保留一些程度的雾，显得逼真。因此可以通过引入0到1的$\omega$对预估透射率进行修正：

$$\tilde{t}(x) = 1- \omega\min_{y\in \Omega(x)}(\min_{c}\frac{I^{c}(y)}{A^{c}})$$

以上的推导过程均假设大气光值A是已知的，在实际上，可以借助暗通道图从原始雾图中求取。具体步骤如下：

1. 求取暗通道图，在暗通道图中按照亮度大小提取最亮的前0.1%的像素
2. 在原始雾图$I(x)$中找到对应位置上具有最高亮度的点的值，作为大气光值A

此外，由于透射率$t$偏小时，会造成$J$偏大，恢复的无雾图像整体向白场过渡，因此需要设定下限值$t_0$，将式子代入整理可得图像恢复公式为：
$$J(x) = \frac{I(x)-A}{\max(t(x), t_0)} + A$$

### SIFT算法

尺度不变特征转换即SIFT是一种常见的计算机视觉的算法，用于检测与描述图像中的局部特征。SIFT特征是基于物体上一些局部外观的兴趣点，与图像的大小、旋转等无关。对于光线、噪声等鲁棒性较强。在学习SIFT算法之前，需要清楚什么是SIFT关键点以及什么SIFT尺度空间。

#### SIFT关键点

SIFT关键点指的是图像中的一些十分突出的点，这些点不会因光照或者旋转等图像改变而消失。比如角点、边缘点、暗区域的亮点以及亮区域的暗点。既然两幅图像中有相同的景物，那么使用某种方法分别提取各自的稳定点，这些点之间会有相互对应的匹配点。所谓关键点，就是在不同的尺度空间的图像下检测出具有方向信息的局部极值点。这些特征点具有三个特征：尺度、方向、大小。


#### 尺度空间

![image](https://user-images.githubusercontent.com/27406337/153530380-53548d46-edec-4c97-948f-335687de5ea4.png)


#### 算法实现

1. **图像高斯金字塔**

图像高斯金字塔实际上是一种图像的尺度空间（分线性和非线性空间），尺度概念用来模拟观察者距离物体的远近程度，在模拟物体远近的同时，还得考虑物体的粗细程度。高斯金字塔的创建过程如下图所示：

![image](https://user-images.githubusercontent.com/27406337/153530620-7def44f0-6169-4f6b-8302-ad904975d896.png)

![image](https://user-images.githubusercontent.com/27406337/153532012-164e76b7-c3ae-4354-967a-d5ebe39513bf.png)

参数：

- 金字塔组数：一般情况下取$\lfloor \log_{2}(\min(M, N))\rfloor - 3$或者$\lfloor \log_{2}(\min(M, N))\rfloor - 4$
- 每层组数：$S_1 = s+3$，s为极值检测需要的层数，一般取3到5。假设高斯金字塔每组有S=5层，则高斯差分金字塔有S-1=4。那么只能在高斯差分金字塔每组的中间两层图像求极值，所以n=2。
- $k = 2^{\frac{1}{s}}$，$\sigma_0=1.6$

具体的生成过程可以参考：https://blog.csdn.net/lhanchao/article/details/52345845

2. **关键点检测**

 关键点是由DOG空间的局部极值点组成的，关键点的初步探查是通过同一组内各DoG相邻两层图像之间比较完成的。为了寻找DoG函数的极值点，每一个像素点要和它所有的相邻点比较，看其是否比它的图像域和尺度域的相邻点大或者小。如图下图所示，中间的检测点和它同尺度的8个相邻点和上下相邻尺度对应的9×2个点共26个点比较，以确保在尺度空间和二维图像空间都检测到极值点。

![image](https://user-images.githubusercontent.com/27406337/153533550-58f98670-0574-4569-a41e-1d47198e2384.png)

以上方法检测到的极值点是离散空间的极值点，以下通过拟合三维二次函数来精确确定关键点的位置和尺度，同时去除低对比度的关键点和不稳定的边缘响应点(因为DoG算子会产生较强的边缘响应)，以增强匹配稳定性、提高抗噪声能力。离散空间的极值点并不是真正的极值点，下图显示了二维函数离散空间得到的极值点与连续空间极值点的差别。利用已知的离散空间点插值得到的连续空间极值点的方法叫做子像素插值。
![image](https://user-images.githubusercontent.com/27406337/153533673-821019ef-9df4-4575-ad3b-12bd5249af1d.png)

可以参考：https://zhuanlan.zhihu.com/p/343522892

3. **关键点的方向**

统计以特征点为圆心，以该特征点所在的高斯图像的尺度的1.5倍为半径的圆内的所有的像素的梯度方向及其梯度幅值，并做1.5σ的高斯滤波(高斯加权，离圆心也就是关键点近的幅值所占权重较高)。
![image](https://user-images.githubusercontent.com/27406337/153533784-94680923-6fd6-487e-8f26-8cc91f0eeeef.png)


4. **构建关键点描述符**

上述过程，只是找到关键点并确定了其方向，但SIFT算法核心用途在于图像的匹配，我们需要对关键点进行数学层面的特征描述，也就是构建关键点描述符。详情可以参考：https://zhuanlan.zhihu.com/p/343522892 。


### HOG算法

### Python部分

关于Python部分算法在此不再做详细的介绍，在每个文件夹里面都有相应ReadMe文件做介绍以及相关的代码。在此只记录实现的算法。

- [x]  ImageSimilarity：图像匹配算法，包括相似性度量的感知哈希算法、模板匹配算法、银行卡号识别实战。
- [x]  ImageSegment：图像分割算法
- [x]  ImageMask：Mask掩膜操作 
- [x]  ImageBlur：滤波
- [x]  ImageGray：图像灰度化操作
- [ ]  ImageMorphology：图像形态学变化
