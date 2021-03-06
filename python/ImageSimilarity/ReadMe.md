## 感知哈希算法

感知哈希算法（Perceptual hash algorithm）对每张图片生成一个“指纹”字符串，然后比较每个图片的指纹。结果越接近说明图像越相似。代码：```PerceptualHash.py```

算法步骤：
1. 对图像进行缩放到8*8的尺寸，目的是为了去除图像的细节，只保留结构、明暗等基本信息，摒弃不同尺寸、比例带来的图像差异；
2. 简化色彩：将缩小后的图片转为64级灰度；
3. 计算平均值：计算所有64个像素的灰度平均值；
4. 比较像素的灰度平均值：将每个像素的灰度与平均值进行比较，大于或等于平均值，记为1；否则为0；
5. 计算哈希值：将上述结果组合一起，构成64位整数，形成“指纹”。得到指纹以后，就可以对比不同的图片，看看64位中有多少位是不一样的。在理论上，这等同于计算"汉明距离"（Hamming distance）。如果不相同的数据位不超过5，就说明两张图片很相似；如果大于10，就说明这是两张不同的图片。

## 模板匹配算法


模板是一副已知的小图像，而模板匹配就是在一副大图像中搜寻目标，已知该图中有要找的目标，且该目标与模板有相同的尺度、方向和图像元素，通过一定的算法可以在图像中找到目标。模板匹配和卷积原理很像，模板在原图像上开始滑动，计算模板与图像被模板覆盖的地方的差别程度，这个差别程度的计算方法在opencv中有6种，然后将每次计算的结果放入一个矩阵里，作为结果输出。加入原图像是A\*B大小，而模板是a\*b大小，则输出结果的矩阵是(A-a+1)\*(B-b+1)。代码：```TemplateMatch.py```

cv.matchTemplate用法：
```
def matchTemplate(image, templ, method, result=None, mask=None): # real signature unknown; restored from __doc__
    """
    matchTemplate(image, templ, method[, result[, mask]]) -> result
    .   @brief Compares a template against overlapped image regions.
    .  
    .   The function slides through image , compares the overlapped patches of size \f$w \times h\f$ against
    .   templ using the specified method and stores the comparison results in result . Here are the formulae
    .   for the available comparison methods ( \f$I\f$ denotes image, \f$T\f$ template, \f$R\f$ result ). The summation
    .   is done over template and/or the image patch: \f$x' = 0...w-1, y' = 0...h-1\f$
    .  
    .   After the function finishes the comparison, the best matches can be found as global minimums (when
    .   #TM_SQDIFF was used) or maximums (when #TM_CCORR or #TM_CCOEFF was used) using the
    .   #minMaxLoc function. In case of a color image, template summation in the numerator and each sum in
    .   the denominator is done over all of the channels and separate mean values are used for each channel.
    .   That is, the function can take a color template and a color image. The result will still be a
    .   single-channel image, which is easier to analyze.
    .  
    .   @param image Image where the search is running. It must be 8-bit or 32-bit floating-point.
    .   @param templ Searched template. It must be not greater than the source image and have the same
    .   data type.
    .   @param result Map of comparison results. It must be single-channel 32-bit floating-point. If image
    .   is \f$W \times H\f$ and templ is \f$w \times h\f$ , then result is \f$(W-w+1) \times (H-h+1)\f$ .
    .   @param method Parameter specifying the comparison method, see #TemplateMatchModes
    .   @param mask Mask of searched template. It must have the same datatype and size with templ. It is
    .   not set by default. Currently, only the #TM_SQDIFF and #TM_CCORR_NORMED methods are supported.
    """
    pass
```

下面对模板匹配方法进行解释：

- cv2.TM_CCOEFF：系数匹配法，计算相关系数，计算出来的值越大，越相关
- cv2.TM_CCOEFF_NORMED：相关系数匹配法，计算归一化相关系数，计算出来的值越接近1，越相关
- cv2.TM_CCORR：相关匹配法，计算相关性，计算出来的值越大，越相关
- cv2.TM_CCORR_NORMED：归一化相关匹配法，计算归一化相关性，计算出来的值越接近1，越相关
- cv2.TM_SQDIFF：平方差匹配法，计算平方不同，计算出来的值越小，越相关
- cv2.TM_SQDIFF_NORMED：归一化平方差匹配法，计算归一化平方不同，计算出来的值越接近0，越相关

### 模板匹配多个对象

在图像中存在多个模板相似对象时，因为模板匹配里的每一处和模板进行对比，所以在同一个模板下，多对象匹配情况下，结果矩阵会有多个值，和最大（小）值接近。如果设置一个阈值，在这个阈值以上或以下的值都被提取出来，再分别得到相应的坐标，则可以进行多模板对象提取，但是这个前提是阈值要设置得当。代码：```multiTemplateMatch.py```


## 实战：信用卡数字识别

信用卡数字识别思路：从信用卡照片中找到待识别的ROI区域，然后将该ROI区域里面的数字与模板一个个进行匹配，得出最优的数字。操作步骤主要分为模板处理以及信用卡图像处理。代码：```creditCardRecog.py```

### 模板处理

模板的照片如下：

![image](https://user-images.githubusercontent.com/27406337/163536871-34dfa432-04e1-4716-add3-862b78be44e7.png)


最终我们需要得到的是每个数字的轮廓，然后做对比，结果如下所示：

![image](https://user-images.githubusercontent.com/27406337/163537055-bbc15d41-5430-4533-b01c-dd15bfb17dc2.png)


#### 信用卡图像处理

1. 灰度化

![image](https://user-images.githubusercontent.com/27406337/163537459-b9584562-093e-4977-8848-16137b1ed60b.png)

2. 顶帽处理

![image](https://user-images.githubusercontent.com/27406337/163537495-0906dd0f-e57b-42f7-b155-df5f33a0b743.png)

3. sobel算子X方向边缘检测

![image](https://user-images.githubusercontent.com/27406337/163537530-9bab0f98-8eb7-4e98-9645-b5764a9b2cee.png)

![image](https://user-images.githubusercontent.com/27406337/163537556-500d2940-6c95-4f37-92dc-b998731d9bb3.png)


4. 膨胀腐蚀

![image](https://user-images.githubusercontent.com/27406337/163537594-dfc551c5-850d-47a3-b9d4-dd0b8953573f.png)


5. 二值化

![image](https://user-images.githubusercontent.com/27406337/163537613-76cb3076-4b7c-4573-a9e4-1eab70d2f0bd.png)

![image](https://user-images.githubusercontent.com/27406337/163537627-ecd28f9f-e91b-40a0-a905-e708552d4715.png)

6. 找到ROI

![image](https://user-images.githubusercontent.com/27406337/163537648-fc5a9f24-5621-4163-969d-7f2a8e2d4b56.png)

7. 识别

![image](https://user-images.githubusercontent.com/27406337/163537668-602ca74c-6e17-4bf8-ba2c-14e5b2722189.png)

