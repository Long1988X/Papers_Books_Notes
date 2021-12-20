[toc]

# ORB-SLAM2: an Open-Source SLAM System for Monocular, Stereo and RGB-D Cameras

## Abstract

1. **ORB-SLAM2**, a complete SLAM system for **monocular, stereo and RGB-D cameras**, including **map reuse**, **loop closing** and **relocalization** capabilities; 

   > ORB-SLAM2是一套基于单目、双目和RGB-D摄像头的完整方案，包括地图重用、回环检测和重定位功能；

2. **Back-End**, based on **Bundle Adjustment**, includes a **lightweight localization mode**, allow for zero-drift localization;

   > 后端，采用基于BA的优化方法，内部包含了一个轻量级的重定位模式，利用VO追踪未建图区域和与地图点匹配，以实现零漂移定位； 

## I. Introduction

1. Cameras are **cheap** and provide **rich information**.

2. **Monocluar SLAM** suffers from **scale drift** and may fail if performing **pure rotations** in exploration.

   > 单目SLAM存在尺度漂移，且在只有纯旋转的操作下，可能会失败；

3. In this paper, propose ORB-SLAM2 with the following contributions:

   * 第一个基于单目、双目和RGB-D摄像头的开源SLAM系统，包括回环检测、重定位和地图重用；
   * RGB-D结果说明，BA优化方法比ICP或者广度和深度最小误差法，更准确；
   * 通过使用近处和远处双目匹配点，和单目观测值，双目的结果比直接双目SLAM方法更准确；（不大明白，啥是直接双目SLAM方法？）
   * 在地图失效的情况，提出了一个轻量级重定位模式，能够更加有效的重用地图；

## II. Related Work

> 介绍了关于双目SLAM和RGB-D SLAM的相关工作； 

### A. Stereo SLAM

### B. RGB-D SLAM

## III. ORB-SLAM2

![image-20211220102740906](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220102740906.png)

ORB_SLAM2由3+1个平行线程组成，包括跟踪、局部建图、回环检测以及在回环检测后的全局BA优化。

之所以说是3+1，因为第四个线程仅在回环检测并确认后才执行。

> a、跟踪：通过寻找对局部地图特征进行匹配，利用纯运动BA最小化重投影误差进行定位每帧图片的相机；
>
> b、局部建图：通过执行局部BA管理局部地图并优化；
>
> c、回环检测：检测大的环并通过执行位姿图优化更正漂移误差。这个线程触发第四线程；
>
> d、全局BA：在位姿图优化之后，计算整个系统最优结构和运动结果。

### A. Monocluar, Close Stereo and Far Stereo Keypoints

> 1. ORB-SLAM2是一种基于特征提取的方法； 
> 2. **近双目特征点的定义**：匹配的**深度值小于40倍**双目或者RGB-D的**基线**，否则的话，是远特征点。
> 3. 近特征点：能够由一帧图像直接三角测量化，得到的深度是精确的估计，并且能够提供尺度，平移和旋转的信息。
> 4. 远特征点：仅仅能够**提供精确的旋转信息**，但是很少的尺度和平移信息。当提供多视图的时候，我们才能三角化那些远的点。

### B. System Bootstrapping

> 使用双目以及RGBD相机的优点在于：
>
> a、直接获得深度信息，而不需像单目一样需做SFM初始化；
>
> b、系统初始化更方便，首帧即为关键帧，且由于知道深度信息，可以从立体点中直接构建初始化地图。

### C. Bundle Adjustment with Monocular and Stereo Constraints

> BA主要用在以下几个地方：
>
> a. 在跟踪线程中使用BA优化相机位姿（纯运动BA）；
>
> b. 优化关键帧的本地窗口和局部建图线程中的点（局部BA）；
>
> c. 在闭环检测后优化所有关键帧和点（全局BA）。
>
> 1. **Motion-only BA**
>
>    > 1. 通过最小化重投影误差的方法，优化的是相机的R、t；
>    > 2. 通过最小化3D点$X^i$与图像上的关键点xi之间的重投影误差；
>    > 3. ρ这里表示鲁棒核函数，鲁棒核函数主要作用是平衡误差不让二范数误差增长过快，π（.）类似于相机内参比上一个尺度，将3D转为2D；
>
>    ![image-20211220110213982](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220110213982.png)
>
>    ![image-20211220110226644](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220110226644.png)
>
> 2. **Local BA**
>
>    > 1. 局部BA相较于纯运动BA，增加了优化该点在地图中的**3D坐标**，即优化了一些主要的**关键帧**以及关键帧中可见的**所有点**；
>
>    ![image-20211220110716149](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220110716149.png)
>
> 3. **Full BA**
>
>    > 全局BA是局部BA的特殊形式，即**所有的关键帧**和**地图点**都要被优化。

### D. Loop Closing and Full BA

> 1. 回环检测主要分两步：
>
>    a.回环信息的检测与确认；
>
>    b.修正和优化位姿图。
>
> 2. 全局BA是一个独立的线程，这使得允许系统在BA优化的同时进行建图和回环检测环节；
>
>    但这会**影响**BA优化与当前地图状态合并，如其他线程运行中产生的新的关键帧与关键点以及在BA过程中回环检测线程检测到了新的环等问题。
>
> 3. 当检测到新的回环时，将终止当前的全局BA优化，因为新的回环会引发新的回环检测，所以把这次的停了也无所谓。

### E. Keyframe Insertion

![image-20211220113720130](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220113720130.png)

> 1. 基本遵循ORB_SLAM方式，即经常插入关键帧并移除多余的关键帧。
>
> 2. 利用远近关键点设置关键帧的插入条件：
>
>    因为已知近关键点相较于远关键点能够提供平移信息，所以需要大量的近关键点用于精确估计平移；
>
>    所以设置**插入帧条件**为：当跟踪的近关键点小于一定数目或者帧中出现的新的近关键点大于一定数目时插入新的关键帧。

### F. Localization Mode

> 1. 定位过程，局部建图与回环检测线程停止，通过追踪线程进行重定位；
> 2. 通过VO匹配图像与地图点云。
> 3. 该方法用于未建图区域时会造成漂移累计，在已知地图中利用点云地图匹配能够实现在地图中的无漂移定位。

## IV. EVALUATION

> 1. Three popular datasets: Kitti, TUM, EuRoC；
> 2. Compared to LSD-SLAM；

![image-20211220134358543](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220134358543.png)

![image-20211220134810017](C:\Users\Pikachu\AppData\Roaming\Typora\typora-user-images\image-20211220134810017.png)

## V. CONCLUSION

