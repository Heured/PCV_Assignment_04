# PCV_Assignment_04
panorama_test


## RANSAC
  "RANSAC(RANdom SAmple Consensus)"随机一致性采样，该方法是用来找到正确模型来拟合带有噪声数据的迭代方法。给定一个模型，例如点集之间的单应性矩阵，RANSAC的基本思想是，数据中包含正确的点和噪声点，合理的模型应该能够在描述正确数据点的同时摒弃噪声点。
  
  给定两个点p1与p2的坐标，确定这两点所构成的直线，要求对于输入的任意点p3，都可以判断它是否在该直线上。初中解析几何知识告诉我们，判断一个点在直线上，只需其与直线上任意两点点斜率都相同即可。实际操作当中，往往会先根据已知的两点算出直线的表达式（点斜式、截距式等等），然后通过向量计算即可方便地判断p3是否在该直线上。
  
  生产实践中的数据往往会有一定的偏差。例如我们知道两个变量X与Y之间呈线性关系，Y=aX+b，我们想确定参数a与b的具体值。通过实验，可以得到一组X与Y的测试值。虽然理论上两个未知数的方程只需要两组值即可确认，但由于系统误差的原因，任意取两点算出的a与b的值都不尽相同。我们希望的是，最后计算得出的理论模型与测试值的误差最小。大学的高等数学课程中，详细阐述了最小二乘法的思想。通过计算最小均方差关于参数a、b的偏导数为零时的值。事实上，在很多情况下，最小二乘法都是线性回归的代名词。
  
    RANSAC的基本假设是：
    (1)数据由“局内点”组成，例如：数据的分布可以用一些模型参数来解释；
    (2)“局外点”是不能适应该模型的数据；
    (3)除此之外的数据属于噪声。
      局外点产生的原因有：噪声的极值；错误的测量方法；对数据的错误假设。
      RANSAC也做了以下假设：给定一组（通常很小的）局内点，存在一个可以估计模型参数的过程；而该模型能够解释或者适用于局内点。
  
### 示例
  假设观测数据中包含局内点和局外点，其中局内点近似的被直线所通过，而局外点远离于直线。简单的最小二乘法不能找到适应于局内点的直线，原因是最小二乘法尽量去适应包括局外点在内的所有点。相反，RANSAC能得出一个仅仅用局内点计算出模型，并且概率还足够高。但是，RANSAC并不能保证结果一定正确，为了保证算法有足够高的合理概率，我们必须小心的选择算法的参数。
  
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/Ransac04.png)
  
  遗憾的是，最小二乘法只适合与误差较小的情况。试想一下这种情况，假使需要从一个噪音较大的数据集中提取模型（比方说只有20%的数据时符合模型的）时，最小二乘法就显得力不从心了。例如下图，肉眼可以很轻易地看出一条直线（模式），但算法却找错了。
  
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/Ransac02.PNG)
  
### 概述
  RANSAC算法的输入是一组观测数据（往往含有较大的噪声或无效点），一个用于解释观测数据的参数化模型以及一些可信的参数。
  RANSAC通过反复选择数据中的一组随机子集来达成目标。被选取的子集被假设为局内点，并用下述方法进行验证：
  
    (1)有一个模型适应于假设的局内点，即所有的未知参数都能从假设的局内点计算得出。
    (2)用1中得到的模型去测试所有的其它数据，如果某个点适用于估计的模型，认为它也是局内点。
    (3)如果有足够多的点被归类为假设的局内点，那么估计的模型就足够合理。
    (4)然后，用所有假设的局内点去重新估计模型，因为它仅仅被初始的假设局内点估计过。
    (5)最后，通过估计局内点与模型的错误率来评估模型。
    上述过程被重复执行固定的次数，每次产生的模型要么因为局内点太少而被舍弃，要么因为比现有的模型更好而被选用
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/Ransac01.jpg)
  
  RANSAC的优点是它能鲁棒的估计模型参数。例如，它能从包含大量局外点的数据集中估计出高精度的参数。RANSAC的缺点是它计算参数的迭代次数没有上限；如果设置迭代次数的上限，得到的结果可能不是最优的结果，甚至可能得到错误的结果。RANSAC只有一定的概率得到可信的模型，概率与迭代次数成正比。RANSAC的另一个缺点是它要求设置跟问题相关的阀值。
  
  RANSAC只能从特定的数据集中估计出一个模型，如果存在两个（或多个）模型，RANSAC不能找到别的模型。
  [参考1](https://www.cnblogs.com/weizc/p/5257496.html)
  [参考2](http://www.cnblogs.com/xrwang/archive/2011/03/09/ransac-1.html)
  
  
## 图像拼接
  同一位置拍摄的两幅或多幅图像是单应性相关的。我们可以使用该约束将很多图像缝补起来，拼成一个大的图像。
  
  本次实验不需要添加新的工具包，编译器为python3.7
  
### 两图
  
```python

# -*- coding: utf-8 -*-
from pylab import *
from numpy import *
from PIL import Image

# If you have PCV installed, these imports should work
from PCV.geometry import homography, warp
from PCV.localdescriptors import sift
from PCV.tools import imtools

"""
This is the panorama example from section 3.3.
"""

# set paths to data folder
featname = ['./data_test/ZhongShan' + str(i + 1) + '.sift' for i in range(2)]
imname = ['./data_test/ZhongShan' + str(i + 1) + '.jpg' for i in range(2)]

# extract features and match
l = {}
d = {}
for i in range(2):
    sift.process_image(imname[i], featname[i])
    l[i], d[i] = sift.read_features_from_file(featname[i])

matches = {}
for i in range(1):
    matches[i] = sift.match(d[i + 1], d[i])

# visualize the matches (Figure 3-11 in the book)
for i in range(1):
    im1 = array(Image.open(imname[i]))
    im2 = array(Image.open(imname[i + 1]))
    figure()
    sift.plot_matches(im2, im1, l[i + 1], l[i], matches[i], show_below=True)


# function to convert the matches to hom. points
def convert_points(j):
    ndx = matches[j].nonzero()[0]
    fp = homography.make_homog(l[j + 1][ndx, :2].T)
    ndx2 = [int(matches[j][i]) for i in ndx]
    tp = homography.make_homog(l[j][ndx2, :2].T)

    # switch x and y - TODO this should move elsewhere
    fp = vstack([fp[1], fp[0], fp[2]])
    tp = vstack([tp[1], tp[0], tp[2]])
    return fp, tp


# estimate the homographies
model = homography.RansacModel()

fp, tp = convert_points(0)
H_01 = homography.H_from_ransac(fp, tp, model)[0]  # im 0 to 1

# warp the images
delta = 2000  # for padding and translation

im1 = array(Image.open(imname[0]), "uint8")
im2 = array(Image.open(imname[1]), "uint8")
im_12 = warp.panorama(H_01, im1, im2, delta, delta)


figure()
# imshow(array(im_42, "uint8"))
imshow(array(im_12, "uint8"))
axis('off')
savefig("example5.png", dpi=300)
show()

```
#### 室内
  
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/SuShe_pipei.png)
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/SuShe.png)
  
#### 室外
  
(1)
  
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/ZhongShan_pipei.png)
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/ZhongShan.png)
  
(2)
  
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoPing_pipei.png)
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoPing.png)
  
本次结果看起来相对较差，也许是连接处噪点较多导致
  
(3)
  
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/Hu_pipei.png)
![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/Hu.png)
  
### 多图
  
原图为CaoChang1~4
```python
# -*- coding: utf-8 -*-
from pylab import *
from numpy import *
from PIL import Image

# If you have PCV installed, these imports should work
from PCV.geometry import homography, warp
from PCV.localdescriptors import sift
from PCV.tools import imtools


"""
download_path = r"D:\pyCharm\pycharm_workspace\2019-3-31QJPingJie\data_test"  # set this to the path where you downloaded the panoramio images
path = r"D:\pyCharm\pycharm_workspace\2019-3-31QJPingJie\data_test"  # path to save thumbnails (pydot needs the full system path)

# list of downloaded filenames
imlist = imtools.get_imlist(download_path)
nbr_images = len(imlist)

# extract features
featlist = [imname[:-3] + 'sift' for imname in imlist]
for i, imname in enumerate(imlist):
    sift.process_image(imname, featlist[i])
"""


"""
This is the panorama example from section 3.3.
"""

# set paths to data folder
featname = ['./data_test/CaoChang' + str(i + 1) + '.sift' for i in range(4)]
imname = ['./data_test/CaoChang' + str(i + 1) + '.jpg' for i in range(4)]


# extract features and match
l = {}
d = {}
for i in range(4):
    sift.process_image(imname[i], featname[i])
    l[i], d[i] = sift.read_features_from_file(featname[i])

matches = {}
for i in range(3):
    matches[i] = sift.match(d[i + 1], d[i])

# visualize the matches (Figure 3-11 in the book)
for i in range(3):
    im1 = array(Image.open(imname[i]))
    im2 = array(Image.open(imname[i + 1]))
    figure()
    sift.plot_matches(im2, im1, l[i + 1], l[i], matches[i], show_below=True)


# function to convert the matches to hom. points
def convert_points(j):
    ndx = matches[j].nonzero()[0]
    fp = homography.make_homog(l[j + 1][ndx, :2].T)
    ndx2 = [int(matches[j][i]) for i in ndx]
    tp = homography.make_homog(l[j][ndx2, :2].T)

    # switch x and y - TODO this should move elsewhere
    fp = vstack([fp[1], fp[0], fp[2]])
    tp = vstack([tp[1], tp[0], tp[2]])
    return fp, tp


# estimate the homographies
model = homography.RansacModel()

fp, tp = convert_points(1)
H_12 = homography.H_from_ransac(fp, tp, model)[0]  # im 1 to 2

fp, tp = convert_points(0)
H_01 = homography.H_from_ransac(fp, tp, model)[0]  # im 0 to 1

tp, fp = convert_points(2)  # NB: reverse order
H_32 = homography.H_from_ransac(fp, tp, model)[0]  # im 3 to 2
'''
tp, fp = convert_points(3)  # NB: reverse order
H_43 = homography.H_from_ransac(fp, tp, model)[0]  # im 4 to 3
'''
# warp the images
delta = 2000  # for padding and translation

im1 = array(Image.open(imname[1]), "uint8")
im2 = array(Image.open(imname[2]), "uint8")
im_12 = warp.panorama(H_12, im1, im2, delta, delta)

im1 = array(Image.open(imname[0]), "f")
im_02 = warp.panorama(dot(H_12, H_01), im1, im_12, delta, delta)

im1 = array(Image.open(imname[3]), "f")
im_32 = warp.panorama(H_32, im1, im_02, delta, delta)

'''
im1 = array(Image.open(imname[4]), "f")
im_42 = warp.panorama(dot(H_32, H_43), im1, im_32, delta, 2 * delta)
'''

figure()
# imshow(array(im_42, "uint8"))
imshow(array(im_32, "uint8"))
axis('off')
savefig("example5.png", dpi=300)
show()

```
  
遇到问题：当原图排序为从左到右时，所得到的拼接图非常杂乱
  
解决办法：将原图排序由从左到右改为从右到左
  
结果：
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoChang_pipei1.png)
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoChang_pipei2.png)
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoChang_pipei3.png)
  ![emmmm](https://github.com/Heured/PCV_Assignment_04/blob/master/imgToShow/CaoChang.png)

  
  
  

