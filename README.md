# HSV2Gray

@[toc]
## 一、引言

------------

**HSV**(Hue, Saturation, Value)，也称六角锥体模型(Hexcone Model)，是一种较为直观的颜色模型，包含色调(H)，饱和度(S)和明度(V)三个参数。

- 色调 Hue (H)
色调 H 反映所处的光谱颜色的位置，在六角锥体模型中用角度度量，取值范围为 0°～360°。RGB 三个通道的取值范围通常是 0-255，因此 OpenCV、[ncnn](https://github.com/Tencent/ncnn) 等库倾向于用 UCHAR 表示颜色。由于 360 超出了 UCHAR 的表示范围，不同的软件在对 H 进行量化时的取值范围有所差异。[OpenCV](https://github.com/opencv/opencv/blob/master/modules/imgproc/src/color_hsv.simd.hpp "OpenCV") 实现了三种范围：0-360， 0-180 和 0-255，本质都是对 H 进行线性变换，将其限制在给定的范围中，cv2 默认 H 的取值范围为 0-180。
- 饱和度 Saturation (S)
饱和度 S 表示颜色接近光谱色的程度，为一比例值，通常取值范围为 0%～100%，OpenCV 将 S 的范围扩大至 0-255。
- 亮度 Value (V)
亮度 V 表示色彩的明亮程度，通常取值范围为 0%～100%，OpenCV 同样将 V 的范围扩大至 0-255。
- 颜色对照表

|   |  黑 | 灰 | 白 | 红 | 橙 | 黄 | 绿 | 青 | 蓝 | 紫 |
| ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
| hmin | 0 | 0 | 0 | 0 / 156 | 11 | 26 | 35 | 78 | 100 | 125 |
| hmax | 180 | 180 | 180 | 10 / 180 | 25 | 34 | 77 | 99 | 124 | 155 |
| smin | 0 | 0 | 0 | 43  | 43  | 43 | 43 | 43 | 43 | 43 |
| smax | 255 | 43 | 30 | 255 | 255 | 255 | 255 | 255 | 255 | 255 |
| vmin | 0 | 46 | 221 | 46 | 46 | 46 | 46 | 46 | 46 | 46 |
| vmax | 46 | 220 | 255 | 255 | 255 | 255 | 255 | 255 | 255 | 255 |

本文使用的图片为 **Lenna 测试图**，即雷娜图。雷娜图（Lenna or Lena）是图像处理领域最受欢迎的测试图，图片中的美丽女性为《花花公子》的玩伴女郎雷娜（LenaSoderberg）。

![雷娜图](images/Lenna.png#pic_center =250x250)

文中对 RGB 三个颜色通道、转换后的 HSV 以及灰度图进行可视化，得出 HSV 和 Gray 进行颜色转换的方法。代码已上传至 [Github](https://github.com/Sherry-XLL/HSV2Gray)，如有建议欢迎留言评论。

## 二、为什么需要 HSV ？

------------

尽管 RGB 是电视和数码相机中使用最广泛的图像表示法，但由于颜色之间的高度相关性，在目标识别和颜色提取任务中人们更多地使用 HSV 而不是 RGB。下图显示了 Lenna 图在 R、G、B 三个通道的颜色分割，可以观察到三个通道中的颜色协同变化，暴露了基于 RGB 模式识别的主要困难。

```python
image = cv2.imread("lenna.png")

# cv2 默认为 BGR 格式，plt.imshow 默认为 RGB 格式
(b, g, r) = cv2.split(image)
bgr = cv2.merge([r, g, b])

# show BGR
plt.subplot(2, 2, 1)
plt.imshow(bgr)
plt.axis('off')
plt.title('original image (BGR)')

B = cv2.merge([zero, zero, b])
plt.subplot(2, 2, 2)
plt.imshow(B)
plt.axis('off')
plt.title('B')

G = cv2.merge([zero, g, zero])
plt.subplot(2, 2, 3)
plt.imshow(G)
plt.axis('off')
plt.title('G')

R = cv2.merge([r, zero, zero])
plt.subplot(2, 2, 4)
plt.imshow(R)
plt.axis('off')
plt.title('R')
plt.show()
```

![在这里插入图片描述](images/bgr.png#pic_center =550x550)

## 三、HSV 三个分量表示什么？

------------

HSV 包含色调(H)，饱和度(S)和明度(V)三个参数，下图基于 HSV 分割，将 HSV 三个通道的分量用灰度图进行表示。与 RGB 的颜色通道相比，色调和饱和度通道不受光照条件的影响，从而支持对象边界的有效识别。

```python
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
(h, s, v) = cv2.split(hsv)
```

![在这里插入图片描述](images/hsv.png#pic_center)


其中，Value 分量表示色彩的明亮程度，通常取值范围为 0%～100%，在 ncnn 和 OpenCV 中都将 V 的范围扩大至了 0-255，此时 HSV 的 V 分量图即为 HSV 转换为灰度图的一种粗略做法。

## 四、绘图误区

------------

值得注意的是，RGB 与 HSV 是两种不同的颜色表示方式，转换后每个颜色通道的意义和数值发生了改变，但仍表示原来的图片，即 RGB、HSV 只是颜色的不同表示方式，在 Photoshop 中可以有更直观的认识：

![在这里插入图片描述](images/前景色.jpg#pic_center =450x350)

网络上经常可以看到使用库函数将 RGB 转换为 HSV 表示，然后直接用 imshow 绘制图片，称其为对应的 HSV 图像，这是不对的：

```python
image = cv2.imread("Lenna.png")
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
plt.imshow(hsv)
plt.title('HSV (plt.imshow)')
plt.show()
```

![在这里插入图片描述](images/hsvshow.png#pic_center)


在 Python 中，matplotlib.pyplot 的 imshow 默认所展示图像的三个通道为 R、G 和 B；cv2 的 imshow 默认所展示图像的三个通道为 B、G 和 R。也就是说，imshow 将 HSV 三个通道的分量当作 RGB 或 BGR 的三个通道得到了如上输出，这样得到的图片从理论上来说是没有意义的。

## 五、HSV 和灰度图的转换

------------

尽管将 HSV 的 V 分量分离得到的图片可以作为灰度图，但图像质量较低。为了得到高质量的灰度图，常用的 HSV 转为灰度图的方法是以 RGB 作为中介。RGB 转换为灰度图的常用公式如下：

> Gray = R \* 0.299 + G \* 0.587 + B \* 0.114

![在这里插入图片描述](images/grayscale.png#pic_center)

对比 RGB 值计算得到的灰度图和 HSV 的 V 分量图，不难看出 RGB 公式转换的灰度图质量更高：
![在这里插入图片描述](images/graycmp.png#pic_center)

因此，将 HSV 转换为灰度图的代码与 HSV 转换为 RGB 的代码类似，先将 HSV 按照公式转换为 RGB 的颜色表示，再根据RGB 转换成灰度图像的常用公式计算对应的灰度值即可，即 HSV -> RGB -> Gray，详见 [HSL and HSV](https://en.wikipedia.org/wiki/HSL_and_HSV)，此处不再赘述。

将三通道的彩色图片转换为灰度图后失去了原图片的部分信息，因此无法实现一一对应的双向转换，在灰度图转 HSV 时只能进行近似处理，直接将灰度值赋于 V，然后 H 和 S 取值为 0.

## 六、参考链接

------------

1. [Github 源码](https://github.com/Sherry-XLL/HSV2Gray)
2. [ncnn](https://github.com/Tencent/ncnn)
3. [Lenna](https://en.wikipedia.org/wiki/Lenna)
4. [OpenCV](https://github.com/opencv/opencv/blob/master/modules/imgproc/src/color_hsv.simd.hpp "OpenCV")
5. [HSL and HSV](https://en.wikipedia.org/wiki/HSL_and_HSV)
6. [Add convert color hsv](https://github.com/Tencent/ncnn/pull/3119)
7. [A high performance terabyte-order RGB to HSV parallel conversion implementation](https://www.tec.ac.cr/sites/default/files/media/doc/final_report_2015_2.pdf)