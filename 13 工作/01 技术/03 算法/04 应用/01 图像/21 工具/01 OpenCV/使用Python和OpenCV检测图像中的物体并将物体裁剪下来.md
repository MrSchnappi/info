
# 使用 python 和 OpenCV 检测图像中的物体并将物体裁剪下来

这个还是会用到的，比如，我之前做的项目，图片很大，如果直接放到 yolo 里跑，resize 到 (416,416)，感觉很多小的地方都够呛能够识别出来。

怎么办呢？之前的图片中有比较大的一部分的黑色边缘，因此，就想把这一部分的黑色边缘去掉。

那么怎么去掉呢？


原图片举例（将红色矩形框部分裁剪出来））：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/gh4F6EDbib.png?imageslim">
</p>



## step1：加载图片，转成灰度图

```
image = cv2.imread("353.jpg")
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
```

## step2: 留下具有高水平梯度和低垂直梯度的图像区域

用 Sobel 算子计算 x，y 方向上的梯度，之后在 x 方向上减去 y 方向上的梯度，通过这个减法，我们留下具有高水平梯度和低垂直梯度的图像区域。

<span style="color:red;">为什么要这么做？原因是什么？</span>

```
gradX = cv2.Sobel(gray, ddepth=cv2.cv.CV_32F, dx=1, dy=0, ksize=-1)
gradY = cv2.Sobel(gray, ddepth=cv2.cv.CV_32F, dx=0, dy=1, ksize=-1)

# subtract the y-gradient from the x-gradient
gradient = cv2.subtract(gradX, gradY)
gradient = cv2.convertScaleAbs(gradient)
```

执行完这一步，得到的图像如下：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/l8e7LfJLC7.png?imageslim">
</p>

## step3：去除图像上的噪声。

首先使用低通滤泼器平滑图像（9x9内核），这将有助于平滑图像中的高频噪声。低通滤波器的目标是降低图像的变化率。如将每个像素替换为该像素周围像素的均值。这样就可以平滑并替代那些强度变化明显的区域。

<span style="color:red;">嗯，是的。这样的确是可以去除噪声的。为什么是高频噪声？有低频噪声吗？</span>

然后，对模糊图像二值化。梯度图像中不大于 90 的任何像素都设置为 0（黑色）。 否则，像素设置为 255（白色）。

```
# blur and threshold the image
blurred = cv2.blur(gradient, (9, 9))
(_, thresh) = cv2.threshold(blurred, 90, 255, cv2.THRESH_BINARY)
```

执行完这一步，得到的图像如下：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/jF1Ef0dcaI.png?imageslim">
</p>

## step4: 用白色填充昆虫身体的空白区域

在上图中我们看到蜜蜂身体区域有很多黑色的空余，我们要用白色填充这些空余，使得后面的程序更容易识别昆虫区域，这需要做一些形态学方面的操作。


```
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (25, 25))
closed = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel)
```


处理之后的图像如下：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/IIACE06C7B.png?imageslim">
</p>


## step5: 去掉图上的一些小的白色斑点

从上图我们发现图像上还有一些小的白色斑点，这会干扰之后的昆虫轮廓的检测，要把它们去掉。

分别执行 4 次形态学腐蚀与膨胀。

```
# perform a series of erosions and dilations
closed = cv2.erode(closed, None, iterations=4)
closed = cv2.dilate(closed, None, iterations=4)
```


执行完这步，得到的图形如下：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/8iJFBD3eGm.png?imageslim">
</p>

## step6：找出昆虫区域的轮廓。

`cv2.findContours()` 函数有三个参数：

- 第一个参数是要检索的图片，必须是为二值图，即黑白的（不是灰度图），所以读取的图像要先转成灰度的，再转成二值图，我们在第三步用`cv2.threshold()`函数已经得到了二值图。
- 第二个参数表示轮廓的检索模式，有四种：
    - `cv2.RETR_EXTERNAL` 表示只检测外轮廓
    - `cv2.RETR_LIST` 检测的轮廓不建立等级关系
    - `cv2.RETR_CCOMP` 建立两个等级的轮廓，上面的一层为外边界，里面的一层为内孔的边界信息。如果内孔内还有一个连通物体，这个物体的边界也在顶层。
    - `cv2.RETR_TREE` 建立一个等级树结构的轮廓。
- 第三个参数为轮廓的近似方法
    - `cv2.CHAIN_APPROX_NONE` 存储所有的轮廓点，相邻的两个点的像素位置差不超过 1，即 `max(abs（x1-x2)，abs(y2-y1))==1`
    - `cv2.CHAIN_APPROX_SIMPLE` 压缩水平方向，垂直方向，对角线方向的元素，只保留该方向的终点坐标，例如一个矩形轮廓只需 4 个点来保存轮廓信息


`cv2.findContours()` 函数返回两个值：

- 一个是轮廓本身，
- 还有一个是每条轮廓对应的属性。

`cv2.findContours()` 函数返回第一个值是 list，list 中每个元素都是图像中的一个轮廓，用 numpy 中的 ndarray 表示。每一个 ndarray 里保存的是轮廓上的各个点的坐标。我们把 list 排序，点最多的那个轮廓就是我们要找的昆虫的轮廓。


然后，我们可以通过 `cv2.drawContours` 在图像上绘制轮廓：

- 第一个参数是指明在哪幅图像上绘制轮廓
- 第二个参数是轮廓本身，在 python 中是一个 list
- 第三个参数指定绘制轮廓 list 中的哪条轮廓，如果是 -1，则绘制其中的所有轮廓
- 第四个参数是轮廓线条的颜色
- 第五个参数是轮廓线条的粗细


`cv2.minAreaRect()` 函数:

主要求得包含点集最小面积的矩形，这个矩形是可以有偏转角度的，可以与图像的边界不平行。

```
(cnts, _) = cv2.findContours(closed.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
c = sorted(cnts, key=cv2.contourArea, reverse=True)[0]

# compute the rotated bounding box of the largest contour
rect = cv2.minAreaRect(c)
box = np.int0(cv2.cv.BoxPoints(rect))

# draw a bounding box arounded the detected barcode and display the image
cv2.drawContours(image, [box], -1, (0, 255, 0), 3)
cv2.imshow("Image", image)
cv2.imwrite("contoursImage2.jpg", image)
cv2.waitKey(0)
```

<span style="color:red;">上面这里，他只是选择了第一个找到的矩形，如果项目中有很多物体，遍历这些矩形，都画出来就行。</span>

执行完这步得到的图形如下：


<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/hEDgFHhlIl.png?imageslim">
</p>


## step7：裁剪。

box里保存的是绿色矩形区域四个顶点的坐标。我将按下图红色矩形所示裁剪昆虫图像。找出四个顶点的 x，y坐标的最大最小值。新图像的高=maxY-minY，宽=maxX-minX。

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/iIC3CAfaIl.png?imageslim">
</p>

```
Xs = [i[0] for i in box]
Ys = [i[1] for i in box]
x1 = min(Xs)
x2 = max(Xs)
y1 = min(Ys)
y2 = max(Ys)
hight = y2 - y1
width = x2 - x1
cropImg = image[y1:y1+hight, x1:x1+width]
```


裁剪出的图片如下：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/181027/5Bbah84aJL.png?imageslim">
</p>





# 相关

- [使用 python 和 OpenCV 检测图像中的物体并将物体裁剪下来](https://blog.csdn.net/liqiancao/article/details/55670749)
9)
9)
