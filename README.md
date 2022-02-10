# 计算机视觉

本项目基于C++实现计算机视觉中的算法。包括传统的计算机视觉算法以及基于深度学习的算法。主要开发工具是visual studio 2019，后续会学习相关的CMake操作。

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

## CMake编译C++



## 计算机视觉传统算法

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

### HOG算法
