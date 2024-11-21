# RobotV_Guidance
Visual guidance system based on point cloud

# 目录

[toc]



# 一、背景介绍

## 1. 项目概述

**项目主题**是《基于点云的视觉引导系统》。

**功能目标**是基于点云的轨迹引导，即无论待引导物体以何种位姿摆放（要求该位姿在机械臂的行程范围内），视觉系统均能定位到该物体，并引导机械臂按需要的轨迹实现一定的工艺流程（比如鞋底涂胶等）。

**应用场景**是鞋底涂胶等需要轨迹引导的工业现场。



## 2. 产品化

本项目实际引导精度在5mm左右，完全能满足工业现场下cm级的引导需求。

如果需要产品化，还需要做的工作是做开发板与机械臂之间的通讯。

本项目使用的kuka机械臂，`TCP通讯`需要安装库卡官方提供的软件包`EthernetKRL`，而该软件包只有windows版本，没有linux版本。如果想在linux下实现通讯，需要插相应的扩展板卡，走`PROFINET通讯`。由于时间和条件有限，本项目没有做开发板与机械臂之间的通讯，而是将开发板算出的结果显示在屏幕上，手工通过机器人示教器写入。如果是类似UR机械臂等原生支持`TCP通讯`的，只要写个简单的socket程序即可，并不需要额外的硬件，在linux下实现起来也较为容易，没有太多限制。



## 3. 未来市场潜力

近些年来，随着机器视觉和工业机器人得到越来越广泛的使用，机器换人的趋势日趋明显。工业机器人可以准确高效地完成重复性的工作，但缺乏柔性，故需要添加机器视觉系统加以辅助。而2D图像缺乏深度信息，常常难以满足六自由度的引导要求，故需要3D点云作为数据来源。

本项目《基于点云的视觉引导系统》，就是基于奥比中光的Astra Pro深度相机和Zora P1开发板，实现基于点云的轨迹引导。轨迹引导已广泛应用于汽车玻璃涂胶、鞋底涂胶等各个领域，未来市场潜力巨大。

此外，基于本项目使用引导方法的扩展，也可实现零件抓取、零件装配等更多的引导需求，本项目只是受限于手头的条件（没有与机械臂匹配的夹爪），没有进行相关的尝试，但原理层面是共通的。



# 二、设备使用情况

| 编号 | 设备                        | 说明                         |
| ---- | --------------------------- | ---------------------------- |
| 1    | 奥比中光的Astra Pro深度相机 | 使用，作为采集点云的传感器   |
| 2    | 奥比中光的Zora P1开发板     | 使用，作为处理点云的计算设备 |
| 3    | 库卡的R2700六轴工业机器人   | 使用，作为实施引导的执行机构 |



# 三、系统架构

系统构成及实物图如下：

| 序号 | 构成       | 选型                        | 备注                       |
| :--- | :--------- | --------------------------- | -------------------------- |
| 1    | 点云相机   | 奥比中光的Astra Pro深度相机 |                            |
| 2    | 开发板     | 奥比中光的Zora P1开发板     | 也可使用普通的PC           |
| 3    | 机械臂     | KUKA KR 210 R2700 extra     | 也可使用其他六轴工业机器人 |
| 4    | 引导设备   | M8螺栓                      | 条件有限，用螺栓代替胶枪   |
| 5    | 待引导物品 | 鞋垫                        |                            |

![实物图_所有物品](assets/实物图_所有物品.jpg)




# 四、部署环境

本项目《基于点云的视觉引导系统》，部署在**奥比中光的Zora P1开发板**和**KUKA的六轴工业机器人**上。

- 奥比中光的Zora P1开发板：板子上跑的是armbian操作系统，部署的是点云采集和点云匹配程序，点云采集采用C++编写，基于奥比中光官方提供的OpenNI2 SDK，详见`3代码&数据/3视觉引导/1获取点云程序`。点云匹配采用python编写，基于open3d库，详见`3代码&数据/3视觉引导/2手眼标定&点云匹配程序`。
- KUKA的六轴工业机器人：部署的是机器人执行程序，详见`3代码&数据/3视觉引导/3kuka机器人程序`。

值得注意的是，由于OpenNI2 SDK和open3d库是跨平台的，所以部署在Zora P1开发板的armbian上的程序，也可以部署在Zora P1开发板的Ubuntu或Android上，或者是windows平台上。



# 五、方案材料

## 1. 系统构成

系统包括：点云相机、开发板、机械臂、引导设备、待引导物品。

详见：[三、系统架构](# 三、系统架构)



## 2. 关键技术创新点

本项目的难点与创新点主要在于，**怎么在各方面条件受限的情况下，尽量提升最后的引导精度**。



条件受限包括：

1. Astra Pro深度相机由于MX400芯片的原因，无法导出相机参数，即无法得到出厂时的准确内参及外参。
2. 手头没有高精度的标定板、没有红外光源，难以重新标定Astra Pro深度相机。
3. Astra Pro深度相机的点云精度（3mm @ 1m）远没有工业级点云相机（0.2mm @ 1m）高。



技术创新点：

### 相机标定

相机标定是视觉系统的基础，工业级的相机标定需要碳纤维（或者玻璃等）的工业级标定板，保证平整度和角点精度。同时需要遮住激光器，并使用红外光源，使得红外相机能采集到清晰的标定板图像。

但是，普通开发者通常不具备上述条件，面临的情况常常是没有标定板和红外光源。为此，本项目使用自制标定板，即通过代码生成高分辨率的棋盘格图像，并用打印机将其打印出来，贴在平板上。但是由于没有红外光源，红外相机只能借助带激光散斑的激光器的光源来拍摄标定板图像，带来的问题是部分角点检测的误差较大。

为了解决这个问题，本项目采用先执行一次相机标定，**保留重投影误差小的70%的点**，再执行一次相机标定。这么做可以明显降低重投影误差、提高精度，使用此方法标定出的相机内外参通过深度图和彩色图的对齐来验证，确实取得了良好的效果。



### 手眼标定

对于眼在手外的情况，即相机固定在机械臂外部，不随机械臂运动，工业场景下的常用手眼标定方法是把标定板固定在机械臂末端，机械臂带着标定板运动多个姿态，固定在机械臂外某处的相机拍摄每个姿态下标定板的图像，并记录机械臂末端位姿。可以建立闭环运动链方程，求解$AX=XB$的方程，A与相机相关，B与机械臂相关，X为手眼矩阵，表示**相机坐标系**到**机械臂基坐标系**的变换关系。

但是，由于缺乏红外光源和高精度标定板，无法从带激光散斑的红外图像中准确提取标定板角点，上述常用的手眼标定方法难以实施，故本项目采取一种更为直接的手眼标定方法。具体做法是：

- 找到一个有4个顶点的物体（如包装盒），用深度相机扫描该物体获得点云，获取4个顶点在**点云相机坐标系**下的坐标；
- 把机械臂的TCP（Tool Center Point，工具中心点）做到引导设备末端，用引导设备末端去触碰那4个顶点，获取4个顶点在**机械臂基座坐标系**下的坐标；
- 通过4个顶点在点云相机坐标系下的坐标、机械臂基座坐标系下的坐标，求解出**点云相机坐标系**到**机械臂基坐标系**的变换关系，完成手眼标定。



### 引导思路

本项目实现的是基于点云的轨迹引导，可以拆解为两个过程：

- 过程一是定位，即视觉系统**定位**到待引导物体；
- 过程二是引导，即视觉系统**引导**机械臂按需要的轨迹实现一定的工艺流程。

考虑到Astra Pro深度相机的点云精度远没有工业级点云相机高，为了尽量减小点云精度对最终引导精度的影响，故本项目使用**相对测量**的思想，将过程一的定位问题转化为点云匹配问题，将过程二的引导问题转化为在模板位置的轨迹基础上做机械臂基坐标系的偏移。



### 点云匹配

常见的点云配准方法有很多，可以分为粗配准和精配准两类，而精配准中的ICP配准结果准确，但依赖比较好的初值。

本项目的预设前提是待引导物体以任意位姿摆放，直接使用ICP很可能会陷入局部最优解，考虑到运行速度与实现难度，故本项目使用**FPFH+RANSAC+ICP**的点云匹配思路，即使用FPFH+RANSAC作为粗配准，获取两个点云之间的粗略变换关系，再以此作为初值，使用ICP作为精配准，获取两个点云之间的精确变换关系。



## 3. 算法设计

### 相机标定部分

使用**张正友的相机标定算法**，但是为了应对红外相机由于激光散斑的影响，导致角点检测精度不高的问题，先执行一次相机标定，**保留重投影误差小的70%的点，再执行一次相机标定**，以提高精度。



相机标定算法，参考`PAMI2000的《A Flexible New Technique for CameraCalibration》`



### 手眼标定部分

把求解**点云相机坐标系**到**机械臂基坐标系**的变换关系，转换为计算两组3D点之间的刚性变换（**umeyama算法**），即通过4个顶点在点云相机坐标系下的坐标、机械臂基座坐标系下的坐标来求解。



umeyama算法，参考`PAMI1991的《Least-squares estimation of transformation parameters between  point patterns》`



### 点云匹配部分

总体思路是，使用**FPFH+RANSAC+ICP**实现点云匹配。用FPFH+RANSAC做粗配准，用ICP做精配准。

使用FPFH作为特征描述子的主要原因是FPFH性能较好且实现简单。FPFH由于使用局部坐标系计算特征，具有平移不变性、旋转不变性。FPFH由于归一化后离散成11等分直方图，对噪声较为鲁棒。

值得注意的是，由于FPFH依赖法线信息，而法线方向具有二义性，故在使用常规方法估算完点云法线后，可以指定一个朝向点，调整点云的法线方向指向该朝向点，使得点云的法线方向是统一的，以提高FPFH的性能。

使用RANSAC是为了实现全局配准，判断条件是配准后点云的重合率等。

以FPFH+RANSAC的粗配准结果，作为ICP精配准的初值，最终获得较高的配准精度。



FPFH算法，参考`ICRA2009的《Fast Point Feature Histograms (FPFH) for 3D Registration》`

RANSAC算法，参考`ACM1981的《Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography》`

ICP算法，参考`PAMI1992的《A Method for Registration of 3D Shapes》`



### 点云匹配的重要优化

值得注意的是，由于点云精度不是很高，加上待引导物体距离机械臂基坐标系的原点较远（近2m），旋转角度的微小误差，经过2m弦长的放大，会对结果造成很大影响。直观理解，$2000×tan(1°)=34.9$，即1°的偏差会造成35mm的影响。

故需要做一个非常重要的处理，**先把待配准点云从机械臂基坐标系转到以点云质心为原点的坐标系，做点云配准，再把点云配准得到的变换关系变换回机械臂基坐标系**。用公式表述为，记机械臂基坐标系到以点云质心为原点的坐标系的变换关系为$T_1$，点云配准得到的变换关系为$T_2$，则最终返回给机械臂基坐标系的偏移量不是$T_2$，而是$inv(T_1)×T_2×T_1$。



### 引导部分

把机械臂引导涂胶的模板位置的轨迹简化为P1到P9这9个点，其中P9与P1重合，如下图所示。如果有需要，可以进一步细化为更多的点。

使用点云匹配的结果修正机械臂的轨迹，具体做法是在模板位置的轨迹基础上做机械臂基坐标系的偏移。

<img src="assets/实物图_鞋垫_标注.jpg" alt="实物图_鞋垫_标注" style="zoom:33%;" />



## 4. 测试结果

鞋垫的模板点云有8927个点，如下图，两点之间的**点间距**大的能达到**近3mm**，故评价点云匹配结果的内点距离阈值取3mm。

![模板点云点间距](assets/模板点云点间距.png)

共进行了12次实验，覆盖平移、旋转等各个位姿。其中，数据1、2、3，为演示视频中的第1、2、3次引导，分别为较大旋转、较大倾斜、旋转180°这3种典型情况。演示视频见`1视频/基于点云的视觉引导系统.mp4`。

实验结果如下表所示，红色为零位的模板点云，黑色为引导位点云，经统计，12组数据在粗配准后的重合率均在33%以上、均值为73%，在**精配准后的重合率均在90%以上**、均值为95%，在**精配准后的内点残差均在1.46以下**、均值为1.33。

而从鞋垫点云中可以看出，点间距在1.5mm到3mm之间，故精配准的结果基本达到了现有条件下的极限。而从实际引导结果来看，12次实验也均是引导成功的。

故本项目《基于点云的视觉引导系统》使用的点云匹配与引导方法是较为成功与实用的，实际引导效果见演示视频：`1视频/基于点云的视觉引导系统.mp4`。

|   编号   |              初始评价              |             粗配准评价             |             精配准评价             |
| :------: | :--------------------------------: | :--------------------------------: | :--------------------------------: |
|  数据1   |  ![guide_1a](assets/guide_1a.png)  |  ![guide_1b](assets/guide_1b.png)  |  ![guide_1c](assets/guide_1c.png)  |
|  重合率  |                0.50                |                0.93                |                0.98                |
| 内点残差 |                1.50                |                1.43                |                1.28                |
|  数据2   |  ![guide_2a](assets/guide_2a.png)  |  ![guide_2b](assets/guide_2b.png)  |  ![guide_2c](assets/guide_2c.png)  |
|  重合率  |                0.09                |                0.89                |                0.97                |
| 内点残差 |                1.72                |                1.52                |                1.32                |
|  数据3   |  ![guide_3a](assets/guide_3a.png)  |  ![guide_3b](assets/guide_3b.png)  |  ![guide_3c](assets/guide_3c.png)  |
|  重合率  |                0.40                |                0.43                |                0.96                |
| 内点残差 |                1.75                |                1.78                |                1.33                |
|  数据4   |  ![guide_4a](assets/guide_4a.png)  |  ![guide_4b](assets/guide_4b.png)  |  ![guide_4c](assets/guide_4c.png)  |
|  重合率  |                0.29                |                0.78                |                0.96                |
| 内点残差 |                1.69                |                1.60                |                1.38                |
|  数据5   |  ![guide_5a](assets/guide_5a.png)  |  ![guide_5b](assets/guide_5b.png)  |  ![guide_5c](assets/guide_5c.png)  |
|  重合率  |                0.20                |                0.82                |                0.93                |
| 内点残差 |                1.56                |                1.65                |                1.37                |
|  数据6   |  ![guide_6a](assets/guide_6a.png)  |  ![guide_6b](assets/guide_6b.png)  |  ![guide_6c](assets/guide_6c.png)  |
|  重合率  |                0.56                |                0.84                |                0.98                |
| 内点残差 |                1.43                |                1.47                |                1.24                |
|  数据7   |  ![guide_7a](assets/guide_7a.png)  |  ![guide_7b](assets/guide_7b.png)  |  ![guide_7c](assets/guide_7c.png)  |
|  重合率  |                0.55                |                0.87                |                0.96                |
| 内点残差 |                1.67                |                1.53                |                1.25                |
|  数据8   |  ![guide_8a](assets/guide_8a.png)  |  ![guide_8b](assets/guide_8b.png)  |  ![guide_8c](assets/guide_8c.png)  |
|  重合率  |                0.28                |                0.79                |                0.95                |
| 内点残差 |                1.76                |                1.67                |                1.37                |
|  数据9   |  ![guide_9a](assets/guide_9a.png)  |  ![guide_9b](assets/guide_9b.png)  |  ![guide_9c](assets/guide_9c.png)  |
|  重合率  |                0.03                |                0.83                |                0.96                |
| 内点残差 |                1.78                |                1.57                |                1.30                |
|  数据10  | ![guide_10a](assets/guide_10a.png) | ![guide_10b](assets/guide_10b.png) | ![guide_10c](assets/guide_10c.png) |
|  重合率  |                0.02                |                0.79                |                0.90                |
| 内点残差 |                1.93                |                1.76                |                1.46                |
|  数据11  | ![guide_11a](assets/guide_11a.png) | ![guide_11b](assets/guide_11b.png) | ![guide_11c](assets/guide_11c.png) |
|  重合率  |                0.10                |                0.49                |                0.91                |
| 内点残差 |                1.71                |                1.70                |                1.45                |
|  数据12  | ![guide_12a](assets/guide_12a.png) | ![guide_12b](assets/guide_12b.png) | ![guide_12c](assets/guide_12c.png) |
|  重合率  |                0.07                |                0.33                |                0.98                |
| 内点残差 |                1.67                |                1.75                |                1.26                |



# 六、研发过程记录

## 1. 上手篇

### 1.1 点云相机上手

本项目使用的点云相机是奥比中光提供的[Astra Pro深度相机](https://developer.orbbec.com.cn/module.html)，原理是基于激光散斑的单目结构光，工作范围是0.6-8m，精度是3mm @ 1m。

<img src="assets/上手_深度相机.jpg" alt="上手_深度相机" style="zoom:33%;" />

浏览3D视觉开发者社区的[开发者中心](https://developer.orbbec.com.cn/develop.html)和[技术文档](https://developer.orbbec.com.cn/technical_library.html)，可以知道Astra Pro深度相机有两套SDK，即OpenNI2 SDK和Astra SDK。相比而言，OpenNI2 SDK更偏向于底层，即获取彩色流、红外流、深度流，而Astra SDK更偏向于上层，除了获取彩色流、红外流、深度流外，还可以获取手部数据流、人体数据流，并支持Unity3D。

需要注意的是，Astra Pro深度相机的彩色流是UVC方式，**无法通过OpenNI2 SDK获取彩色流（可以通过OpenCV获取彩色流），但可以获取深度流**。而使用Astra SDK既可以获取深度流，也可以获取彩色流。



### 1.2 点云相机的工具

从3D视觉开发者社区的[下载中心](https://developer.orbbec.com.cn/download.html?id=31)下载Orbbec Viewer工具，当前的最新版本是`Orbbec Viewer_v1.1.1`，此工具只在windows平台下有，是一个非常好用的可视化工具。可以从界面上轻松看到深度流、彩色流、红外流、点云流，并轻松保存。

![OrbbecViewer](assets/OrbbecViewer.png)



### 1.3 点云相机的SDK

本人主要是在windows平台下使用VS2017做开发，开发完成后再移植到linux下。

参照3D视觉开发者社区的[技术文库](https://developer.orbbec.com.cn/technical_library.html?id=21)的指引，[下载中心](https://developer.orbbec.com.cn/download.html?id=31)下载Astra SDK、OpenNI2 SDK、设备驱动，当前的windows最新版本是`AstraSDK-v2.1.1-vs2015-win64`、`windows_V2.3.0.65`、`SensorDriver_V4.3.0.17`。

先安装驱动，再结合[Astra SDK文档](https://developer.orbbec.com.cn/technical_library.html?id=21)和[OpenNI2 SDK文档](https://developer.orbbec.com.cn/technical_library.html?id=29)熟悉各个API。



#### win10的Astra SDK

对于Astra SDK，解压压缩包后，打开`AstraSDK-v2.1.1-vs2015-win64/samples/vs2015`下的`astra-samples.sln`，即可启动VS2017调试运行例程。

![AstraSDK](assets/AstraSDK.png)



#### win10的OpenNI2 SDK

对于OpenNI2 SDK，解压压缩包后，`OpenNI2-SDK-windows_V2.3.0.65/windows/Samples`下的`samples`和`samples.old`文件夹下各有一个`CMakeLists.txt`文件，使用cmake可以生成VS工程，即可启动VS2017调试运行例程。

![Openni2SDK](assets/Openni2SDK.png)



在众多的例程中，具有较高参考价值的是：

- Astra SDK例程的`InfraredColorReaderEvent`（通过事件使用Astra SDK读取红外和彩色流）
- OpenNI2 SDK的samples.old例程的`GeneratePointCloud`（通过深度帧数据生成点云）
- OpenNI2 SDK的samples.old例程的`SoftD2C`（使用软件D2C的接口实现深度图和彩色图的对齐）



### 1.4 开发板上手

本项目使用的开发板是奥比中光提供的[Zora P1开发板](https://developer.orbbec.com.cn/product_details.html)，内存2GB，存储16GB，板载wifi，默认操作系统是armbian。

|               安装亚克力前               |               安装亚克力后               |
| :--------------------------------------: | :--------------------------------------: |
| ![上手_开发板1](assets/上手_开发板1.jpg) | ![上手_开发板2](assets/上手_开发板2.jpg) |
|                  上电后                  |                 接显示器                 |
| ![上手_开发板3](assets/上手_开发板3.jpg) | ![上手_开发板4](assets/上手_开发板4.jpg) |



当前的linux最新版本的SDK是`AstraSDK-v2.1.1-Linux-aarch64`和`OpenNI-Linux-Arm64-2.3.0.65`。

在桌面上新建名为`SDK`的文件夹，放入下载好的`AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64.tar.gz`和`OpenNI-Linux-Arm64-2.3.0.65.rar`，并解压缩。



#### armbian的Astra SDK

安装Astra SDK：

```shell
cd ~/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/install
sudo sh ./install.sh 
```

返回的提示：

```shell
Linux installer script for Astra SDK

Installing rules for orbbec devices into /etc/udev/rules.d/
Done.

NOTES:
We suggest adding the following lines to your .bash_profile or .bashrc
export ASTRA_SDK_INCLUDE=/home/orbbec/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/install/include
export ASTRA_SDK_LIB=/home/orbbec/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/install/lib
```

根据提示，将最后两行加入环境变量中。此处有一个坑，自动生成的`ASTRA_SDK_INCLUDE`和`ASTRA_SDK_LIB`与实际路径不符，不应该包含`install`目录，此处一定要修改，如下：

```shell
# 打开环境变量文件
sudo nano ~/.bashrc
# 将下面2行加入环境变量的最后
export ASTRA_SDK_INCLUDE=/home/orbbec/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/include
export ASTRA_SDK_LIB=/home/orbbec/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/lib
# 使环境变量立即生效
source ~/.bashrc
```

试验Astra SDK是否安装成功，打开预编译好的`DepthReaderPoll`文件，可以看到对深度流某个像素的深度值是读取并打印成功的：

```shell
cd /home/orbbec/Desktop/SDK/AstraSDK-v2.1.1-24f74b8b15-20200426T012326Z-Linux-aarch64/bin
./DepthReaderPoll
```

![armbian_astrasdk上手](assets/armbian_astrasdk上手.png)



#### armbian的OpenNi2 SDK

参考目录下的`README`文件，安装一些依赖项：

```shell
# 先更新
sudo apt-get update
sudo apt-get upgrade

# 再安装依赖
sudo apt-get install g++
sudo apt-get install python
sudo apt-get install libusb-1.0-0-dev
sudo apt-get install libudev-dev
sudo apt-get install openjdk-8-jdk
sudo apt-get install freeglut3-dev
sudo apt-get install doxygen
sudo apt-get install graphviz
```

安装OpenNi2 SDK：

```shell
cd /home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65
sudo chmod a+x install.sh
sudo ./install.sh
source OpenNIDevEnvironment
```

此处有一个坑，自动生成的`OpenNIDevEnvironment`文件中的`OPENNI2_INCLUDE`与实际路径不符，`Include`应为`include`，此处一定要修改，即把`OpenNIDevEnvironment`的文件内容修改如下：

```sh
export OPENNI2_INCLUDE=/home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65/include
export OPENNI2_REDIST=/home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65/Redist
```

试验OpenNi2 SDK是否安装成功，打开预编译好的`SimpleViewer`文件，可以看到对深度流是可视化成功的：

```shell
cd /home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65/Samples/Bin
sudo chmod a+x SimpleViewer
./SimpleViewer
```

![armbian_openni2sdk上手](assets/armbian_openni2sdk上手.png)



### 1.5 双目相机标定

#### 自制标定板

Astra Pro深度相机由于MX400芯片的原因，无法导出相机参数，即无法得到出厂时的准确内参及外参。而内参和外参关系到深度图转点云和深度图与彩色图对齐，故需要自己做双目相机标定。

由于条件有限，手头没有高精度标定板，故只能自制标定板，通过代码生成棋盘格图片，然后打印出来，贴到一个平整的表面上。

相关代码和供打印的文件见`3代码&数据/1双目相机标定/1制作棋盘格标定板`，其中`main_generate_chessboard.py`是用于生成棋盘格图片的代码，`chessBoard.bmp`是生成的棋盘格图片，`棋盘格.docx`是用于打印的A4纸大小棋盘格的文件。

如下图，棋盘格的宽度有254mm，有10各，则每格为25.4mm。

<img src="assets/chessboard-word.png" alt="棋盘格-word" style="zoom:33%;" />

自制的棋盘格标定板如下图：

<img src="assets/实物图_棋盘格.jpg" alt="实物图_棋盘格" style="zoom:33%;" />



#### 执行双目相机标定

下面执行双目相机标定的过程，在棋盘格整体都在彩色相机和红外相机视野内的情况下，摆放多个姿态，尽量使得棋盘格覆盖视野的各个角落，取20个姿态，执行双目相机标定，标定过程如下图所示。

| 双目相机标定                                                 | 彩色图                         | 红外图                   |
| ------------------------------------------------------------ | ------------------------------ | ------------------------ |
| <img src="assets/实物图_相机标定.jpg" alt="实物图_相机标定" style="zoom:50%;" /> | ![Color_1](assets/Color_1.bmp) | ![IR_1](assets/IR_1.bmp) |

此处最好的情况是，关闭深度相机的激光器，使用外置的红外光源，使用高精度标定板，执行双目相机标定。但由于条件有限，没有红外光源和高精度标定板，故只能使用自制的标定板，采集带激光散斑的红外图像。如下图所示，可以看到由于激光散斑的影响，红外图像有些角点识别错了，故筛除30%的重投影误差大的角点，使用剩下的角点来做双目相机标定。

标定的结果如下表，可以发现，筛除重投影误差大的角点后，总体的重投影误差显著减小了。相关代码和数据见`3代码&数据/1双目相机标定/2执行双目相机标定`。

| 数据                 | 不剔除角点执行双目相机标定                                   | 剔除30%重投影误差大的角点执行双目相机标定                    |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| RGB相机的重投影误差  | 0.217                                                        | 0.145                                                        |
| RGB相机的相机内参    | [[608.10629918   0.         305.86548324]<br/> [  0.         609.76501069 235.87973116]<br/> [  0.           0.           1.        ]] | [[605.38898333   0.         306.20670369]<br/> [  0.         607.6177079  238.36161457]<br/> [  0.           0.           1.        ]] |
| RGB相机的畸变系数    | [[ 0.00946966  0.70169147 -0.00552242 -0.00248239 -1.95620678]] | [[ 0.00285554  0.71769527 -0.00521712 -0.0021043  -1.96128222]] |
| IR相机的重投影误差   | 1.168                                                        | 0.482                                                        |
| IR相机的相机内参     | [[555.37517959   0.         309.76861419]<br/> [  0.         561.84417857 258.47138992]<br/> [  0.           0.           1.        ]] | [[572.35087383   0.         308.60311538]<br/> [  0.         576.61281751 250.66719589]<br/> [  0.           0.           1.        ]] |
| IR相机的畸变系数     | [[-0.15317197  1.02021984 -0.00731694 -0.00666425 -2.13385382]] | [[-0.17546213  1.23176247 -0.00450702 -0.00668543 -2.7598227 ]] |
| 双目相机的重投影误差 | 0.906                                                        | 0.387                                                        |
| 双目相机转换关系的R  | [[ 0.99986445  0.01407785  0.00853784]<br/> [-0.01391202  0.99971923 -0.01918091]<br/> [-0.00880547  0.01905953  0.99977957]] | [[ 0.99985107  0.01379429  0.01037083]<br/> [-0.01372216  0.99988138 -0.00699429]<br/> [-0.01046608  0.00685094  0.99992176]] |
| 双目相机转换关系的t  | [[-25.69905405]<br/> [ -3.41029246]<br/> [-17.07652252]]     | [[-25.37136205]<br/> [ -1.52660596]<br/> [ -4.54607117]]     |

根据重投影误差滤除角点举例：

![相机标定_角点滤除举例_标注](assets/相机标定_角点滤除举例_标注.png)



#### 验证标定精度

使用OpenNI2 SDK的`SoftD2C`例程来验证相机内外参的精度如何，对比的是奥比中光王工提供的另一台Astra Pro深度相机的内外参。固定深度相机与《一往无前》这本书不动，更换相机内外参，**查看书的彩色图与深度图的对齐效果**。可以发现`使用70%重投影误差小的角点`标定出来的相机内外参，明显好于`王工提供的相机内外参`，明显好于`使用所有角点`标定出来的相机内外参。故后续采用`使用70%重投影误差小的角点`标定出来的相机内外参。

| 使用所有角点                                             | 使用70%重投影误差小的角点                            | 王工提供的相机内外参                                 |
| -------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| ![双目相机标定_2不滤点](assets/双目相机标定_2不滤点.png) | ![双目相机标定_3滤点](assets/双目相机标定_3滤点.png) | ![双目相机标定_1王工](assets/双目相机标定_1王工.png) |

王工提供的Astra Pro相机参数：

```cpp
// 王工提供的Astra Pro相机参数
// L是IR图，R是彩色图
// [fx,fy,cx,cy]
cameraParams.l_intr_p[0] = 578.546021;
cameraParams.l_intr_p[1] = 578.546021;
cameraParams.l_intr_p[2] = 309.359985;
cameraParams.l_intr_p[3] = 232.233002;
// [fx, fy, cx, cy]
cameraParams.r_intr_p[0] = 593.426025;
cameraParams.r_intr_p[1] = 593.426025;
cameraParams.r_intr_p[2] = 315.527008;
cameraParams.r_intr_p[3] = 235.903000;
// [r00,r01,r02;r10,r11,r12;r20,r21,r22]
cameraParams.r2l_r[0] = 0.999968;
cameraParams.r2l_r[1] = 0.004491;
cameraParams.r2l_r[2] = -0.006635;
cameraParams.r2l_r[3] = -0.004534;
cameraParams.r2l_r[4] = 0.999969;
cameraParams.r2l_r[5] = -0.006485;
cameraParams.r2l_r[6] = 0.006605;
cameraParams.r2l_r[7] = 0.006515;
cameraParams.r2l_r[8] = 0.999957;
// [t1,t2,t3]
cameraParams.r2l_t[0] = -24.720200;
cameraParams.r2l_t[1] = 0.074482;
cameraParams.r2l_t[2] = -0.168342;
// [k1,k2,p1,p2,k3]
cameraParams.l_k[0] = -0.076802;
cameraParams.l_k[1] = 0.155459;
cameraParams.l_k[2] = 0.000000;
cameraParams.l_k[3] = 0.000367;
cameraParams.l_k[4] = -0.001332;
// [k1,k2,p1,p2,k3]
cameraParams.r_k[0] = 0.104120;
cameraParams.r_k[1] = -0.108380;
cameraParams.r_k[2] = 0.000000;
cameraParams.r_k[3] = -0.001978;
cameraParams.r_k[4] = -0.003020;
```

本人标定出的Astra Pro相机参数：

```cpp
// 本人双目标定出来的参数, 自制标定板, 保留70%的角点, 重投影误差0.387
// L是IR图，R是彩色图
// [fx,fy,cx,cy]
cameraParams.l_intr_p[0] = 572.35087383;
cameraParams.l_intr_p[1] = 576.61281751;
cameraParams.l_intr_p[2] = 308.60311538;
cameraParams.l_intr_p[3] = 250.66719589;
// [fx, fy, cx, cy]
cameraParams.r_intr_p[0] = 605.38898333;
cameraParams.r_intr_p[1] = 607.6177079;
cameraParams.r_intr_p[2] = 306.20670369;
cameraParams.r_intr_p[3] = 238.36161457;
// [r00,r01,r02;r10,r11,r12;r20,r21,r22]
cameraParams.r2l_r[0] = 0.99985107;
cameraParams.r2l_r[1] = 0.01379429;
cameraParams.r2l_r[2] = 0.01037083;
cameraParams.r2l_r[3] = -0.01372216;
cameraParams.r2l_r[4] = 0.99988138;
cameraParams.r2l_r[5] = -0.00699429;
cameraParams.r2l_r[6] = -0.01046608;
cameraParams.r2l_r[7] = 0.00685094;
cameraParams.r2l_r[8] = 0.99992176;
// [t1,t2,t3]
cameraParams.r2l_t[0] = -25.37136205;
cameraParams.r2l_t[1] = -1.52660596;
cameraParams.r2l_t[2] = -4.54607117;
// [k1,k2,p1,p2,k3]
cameraParams.l_k[0] = -0.17546213;
cameraParams.l_k[1] = 1.23176247;
cameraParams.l_k[2] = -0.00450702;
cameraParams.l_k[3] = -0.00668543;
cameraParams.l_k[4] = -2.7598227;
// [k1,k2,p1,p2,k3]
cameraParams.r_k[0] = 0.00285554;
cameraParams.r_k[1] = 0.71769527;
cameraParams.r_k[2] = -0.00521712;
cameraParams.r_k[3] = -0.0021043;
cameraParams.r_k[4] = -1.96128222;
```



## 2. 室内重建篇

### 2.1 概述

在做主体任务`视觉引导`之前，先要测一测Astra Pro深度相机的性能，故做一个简单的`室内重建`。

室内重建的输入是对齐后的彩色图和深度图、相机内参，输出是整个场景的三角面片文件。如下表。

|                       彩色图                       |                        深度图                        |
| :------------------------------------------------: | :--------------------------------------------------: |
|       ![cv_rgb_0001](assets/cv_rgb_0001.png)       | ![aligned_depth_0001](assets/aligned_depth_0001.png) |
|                      手机实拍                      |                       重建结果                       |
| ![室内重建_手机实拍](assets/室内重建_手机实拍.jpg) |  ![室内重建_重建结果](assets/室内重建_重建结果.png)  |



### 2.2 RGBD图采集

OpenNI2 SDK的samples.old的`SoftD2C`的例程实现了深度图和彩色图的对齐，基于其修改成**每隔100ms采集一次图像，并保存对齐后的彩色图和深度图**，相关代码和数据见`3代码&数据/2室内重建/1RGBD图片采集`和`3代码&数据/2室内重建/2室内重建输入数据`。



### 2.3 室内重建

室内重建参考了[open3d的重建系统](https://github.com/intel-isl/Open3D/tree/v0.10.0/examples/Python/ReconstructionSystem)，实现的是`CVPR2015的《Robust Reconstruction of Indoor Scenes》`, 点云配准使用的是`ICCV2017的《Colored Point Cloud Registration Revisited》`。相关代码见`3代码&数据/2室内重建/3室内重建代码`。

算法包含四大步骤：

1. 生成场景片段
2. 配准场景片段
3. 改善配准结果
4. 整合片段并重建场景



此处采集的图片是本人的工位，从室内重建生成的三角面片文件来看，室内重建的效果还是不错的。实际重建结果见`3代码&数据/2室内重建/4室内重建输出结果/室内重建_重建结果.ply`。




## 3. 视觉引导篇

### 3.1 概述

基于奥比中光的Astra Pro深度相机，实现对鞋垫的轨迹引导。

<img src="assets/引导位1修正轨迹.jpg" alt="引导位1修正轨迹" style="zoom: 33%;" />



### 3.2 系统构成

系统包括：点云相机、开发板、机械臂、引导设备、待引导物品。

详见：[三、系统架构](# 三、系统架构)


![实物图_所有物品](assets/实物图_所有物品.jpg)



### 3.3 机器人做TCP

六轴工业机器人的TCP（Tool Center Point，工具中心点）默认在第六轴法兰末端，需要先将其修改到引导设备的末端，即修改到螺栓的末端，如下图。

<img src="assets/实物图_TCP.jpg" alt="实物图_TCP" style="zoom:33%;" />

做TCP常用的方法是4点法，即让TCP以4个姿态去接近同一个点，从而解算出TCP与第六轴末端的转换关系。

| ![做TCP_1](assets/做TCP_1.jpg) | ![做TCP_2](assets/做TCP_2.jpg) |
| :----------------------------: | :----------------------------: |
| ![做TCP_3](assets/做TCP_3.jpg) | ![做TCP_4](assets/做TCP_4.jpg) |

实施步骤如下表：

| 步骤                                                        | 示意图                                                       |
| :---------------------------------------------------------- | :----------------------------------------------------------- |
| 1. 在KUKA机器人的示教器上选择"XYZ 4点法"，进入做TCP的模式。 | <img src="assets/kuka_做TCP_1.jpg" alt="kuka_做TCP_1" style="zoom: 33%;" /> |
| 2. 控制TCP以4个姿态去接近同一个点。                         | ![kuka_做tcp_xyz4点法](assets/kuka_做tcp_xyz4点法.png)       |
| 3. 计算出当前的TCP，存入某个工具坐标系中。                  | <img src="assets/kuka_做TCP_2.jpg" alt="kuka_做TCP_2" style="zoom: 33%;" /> |



### 3.4 手眼标定

固定好深度相机，使其在整个实验过程中不要移动。

准备一个有4个明显顶点且便于深度相机获取点云的物品，如奥比中光深度相机的包装盒。将当前的工具坐标系切换为刚刚做完TCP保存的坐标系，此时机器人示教器上的位姿为螺栓末端的位姿。

![实物图_手眼标定](assets/实物图_手眼标定.jpg)

控制机械臂末端的螺栓分别靠近物品的4个顶点，并记录当前的位姿，再用深度相机获取物品的点云，记录4个顶点的坐标，如下表。

|                实物图                |                   机械臂位姿                   |                      点云坐标                      |
| :----------------------------------: | :--------------------------------------------: | :------------------------------------------------: |
| ![手眼标定_1](assets/手眼标定_1.jpg) | ![kuka_手眼标定_1](assets/kuka_手眼标定_1.jpg) | ![handeye_pc_1_crop](assets/handeye_pc_1_crop.jpg) |
| ![手眼标定_2](assets/手眼标定_2.jpg) | ![kuka_手眼标定_2](assets/kuka_手眼标定_2.jpg) | ![handeye_pc_2_crop](assets/handeye_pc_2_crop.jpg) |
| ![手眼标定_3](assets/手眼标定_3.jpg) | ![kuka_手眼标定_3](assets/kuka_手眼标定_3.jpg) | ![handeye_pc_3_crop](assets/handeye_pc_3_crop.jpg) |
| ![手眼标定_4](assets/手眼标定_4.jpg) | ![kuka_手眼标定_4](assets/kuka_手眼标定_4.jpg) | ![handeye_pc_4_crop](assets/handeye_pc_4_crop.jpg) |

通过4个顶点在点云相机坐标系下的坐标、机械臂基座坐标系下的坐标，求解出点云相机坐标系到机械臂基坐标系的变换关系。值得注意的是，需要将坐标统一到mm单位下。


| 说明                                             | 数值                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 纸盒上表面的4个顶点在**点云相机坐标系**下的坐标  | [[  75.428,  197.79 ,  956.   ],<br/> [  70.142,   81.053,  889.   ],<br/> [-153.098,   89.047,  891.   ],<br/> [-142.923,  210.181,  959.   ]] |
| 纸盒上表面的4个顶点在**机械臂基坐标系**下的坐标  | [[1670.39, -213.59,  833.06],<br/> [1817.12, -213.59,  833.06],<br/> [1814.45,   -1.25,  833.06],<br/> [1667.49,   -1.25,  833.06]] |
| **点云相机坐标系**到**机械臂基坐标系**的变换关系 | [[  -0.03975,   -0.86912,   -0.493  , 2321.88173],<br/> [  -0.99913,    0.04094,    0.00838, -158.65519],<br/> [   0.0129 ,    0.4929 ,   -0.86999, 1565.96284],<br/> [   0.     ,    0.     ,    0.     ,    1.     ]] |




### 3.5 点云匹配&实施引导

准备一个待引导物品，即鞋垫。

![实物图_轨迹引导](assets/实物图_轨迹引导.jpg)

采集一幅点云，根据点云的原点和手眼关系，做可视化，如下图，从直观上验证手眼关系是否正确。

![手眼标定示意图](assets/手眼标定示意图.png)


实施步骤如下表：

| 步骤                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. 扫描零位的鞋垫，滤除无关的点云，作为模板点云。（右图红色） | ![model_crop](assets/model_crop.jpg)                         |
| 2. 把当前基坐标系的值全部赋0。                               | <img src="assets/kuka_设置base_1.jpg" alt="kuka_设置base_1" style="zoom:33%;" /><br /><img src="assets/kuka_设置base_2.jpg" alt="kuka_设置base_2" style="zoom:33%;" /> |
| 3. 在零位制作模板轨迹。                                      | <img src="assets/零位模板轨迹.jpg" alt="零位模板轨迹" style="zoom: 33%;" /> |
| 4. 改变鞋垫的位姿，重新获取当前的鞋垫点云。（右图黑色）      | ![guide_1_crop](assets/guide_1_crop.jpg)                     |
| 5. 执行点云匹配，计算返回给机械臂基坐标系的偏移量，手动通过示教器赋值给当前基坐标系。 | <img src="assets/kuka_设置base_3.jpg" alt="kuka_设置base_3" style="zoom:33%;" /> |
| 6. 机器人执行修正后的轨迹，完成轨迹引导                      | <img src="assets/引导位1修正轨迹.jpg" alt="引导位1修正轨迹" style="zoom: 33%;" /> |

模板轨迹的9个点位与对应的机器人程序如下表：

|                 机器人程序                 |                     对应点位                     |
| :----------------------------------------: | :----------------------------------------------: |
| ![kuka_引导程序](assets/kuka_引导程序.jpg) | ![实物图_鞋垫_标注](assets/实物图_鞋垫_标注.jpg) |



### 3.6 代码运行环境准备

视觉引导相关的代码和数据见`3代码&数据/3视觉引导`。

视觉引导部署在开发板上的代码分为两部分：

- 保存ply格式的点云：在OpenNi2 SDK的例程`SimpleRead`基础上修改的，深度图转点云保存为ply部分参照了例程`GeneratePointCloud`，为c++程序。
- 点云匹配和计算返回机械臂的偏移量：基于open3d库开发的，为python程序。



#### 点云采集

编译代码：

```shell
cd /home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65
source OpenNIDevEnvironment
cd /home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65/Samples/SimpleRead
make
```

运行代码：

```shell
cd /home/orbbec/Desktop/SDK/OpenNI-Linux-Arm64-2.3.0.65/Samples/SimpleRead/Bin/Arm64-Release
./SimpleRead
```

保存点云后，为了可视化查看效果，可以安装meshlab：

```shell
sudo apt-get install meshlab
```

![armbian_视觉引导_点云采集](assets/armbian_视觉引导_点云采集.png)



#### 点云匹配

armbian已经自带了python3（Python3.6.9），但有两个库还需要安装：numpy和open3d。

首先安装pip，并将pip的源换为国内的：

```shell
# 先安装python3的pip
sudo apt-get install python3-pip
# 修改pip源为国内的
cd ~
mkdir .pip
cd .pip
nano pip.conf
```
在pip.conf中输入以下内容

```shell
[global]
timeout = 10
index-url = https://pypi.tuna.tsinghua.edu.cn/simple/
index-index-url = https://pypi.douban.com/simple/
[install]
trusted-host = 
	pypi.tuna.tsinghua.edu.cn
	pypi.douban.com
```

安装numpy，需要自己编译：

```sh
# 先安装python编译包的一些工具
pip3 install setuptools
pip3 install Cython
pip3 install wheel
# 编译安装numpy
pip3 download numpy
unzip numpy-1.19.4.zip 
cd numpy-1.19.4/
python3 setup.py install
```

或者使用另一种方法安装numpy：

```sh
sudo apt-get install python3-numpy
```

安装open3d，参照[官网的指引](http://www.open3d.org/docs/0.11.0/arm.html)，下载源码和依赖的第三方包，自己编译，编译过程中需要联网，要编译安装很久很久：

```shell
# 安装依赖
sudo apt-get update -y
sudo apt-get install -y apt-utils build-essential git cmake
sudo apt-get install -y python3 python3-dev python3-pip
sudo apt-get install -y xorg-dev libglu1-mesa-dev
sudo apt-get install -y libblas-dev liblapack-dev liblapacke-dev
sudo apt-get install -y libsdl2-dev libc++-7-dev libc++abi-7-dev libxi-dev
sudo apt-get install -y clang-7

# 安装虚拟环境
sudo apt-get install -y python3-virtualenv ccache

# 下载源码编译安装
# Optional: create and activate virtual environment
virtualenv --python=$(which python3) ${HOME}/venv
source ${HOME}/venv/bin/activate

# Clone
git clone --recursive https://github.com/intel-isl/Open3D
cd Open3D
git submodule update --init --recursive
mkdir build
cd build

# Configure
# > Set -DBUILD_CUDA_MODULE=ON if CUDA is available (e.g. on Nvidia Jetson)
# > Set -DBUILD_GUI=ON if OpenGL is available (e.g. on Nvidia Jetson)
# > We don't support TensorFlow and PyTorch on ARM officially
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_SHARED_LIBS=ON \
    -DBUILD_CUDA_MODULE=OFF \
    -DBUILD_GUI=OFF \
    -DBUILD_TENSORFLOW_OPS=OFF \
    -DBUILD_PYTORCH_OPS=OFF \
    -DBUILD_UNIT_TESTS=ON \
    -DCMAKE_INSTALL_PREFIX=~/open3d_install \
    -DPYTHON_EXECUTABLE=$(which python) \
    ..

# Build C++ library
make -j$(nproc)

# Run tests (optional)
make tests -j$(nproc)
./bin/tests --gtest_filter="-*Reduce*Sum*"

# Install C++ package (optional)
make install

# Install Open3D python package (optional)
make install-pip-package -j$(nproc)
python -c "import open3d; print(open3d)"
```

numpy和open3d安装成功后，即可运行代码，点云匹配后打印出了返回给机械臂基坐标系的偏移量：

![armbian_视觉引导_点云匹配](assets/armbian_视觉引导_点云匹配.png)



# 七、技术优势分析

技术优势主要在于，怎么在各方面条件受限的情况下，尽量提升最后的引导精度。需要从各个层面去优化，包括：相机标定、手眼标定、引导思路、点云匹配。

详见：[五、方案材料 2. 关键技术创新点](# 五、方案材料)



# 八、基于开发套件产生的bug及建议

## 已知bug和问题记录

1. Astra Pro深度相机无法导出相机参数，即无法得到出厂时的准确内参及外参，原因出在MX400芯片。

2. armbian下的OpenNi2 SDK，即`OpenNI-Linux-Arm64-2.3.0.65.rar`，里面的`install.sh`文件，环境变量`OPENNI2_INCLUDE`中写的是`Include`，与实际路径`include`不符，需要修改过来，否则会引起编译报错。

   ![提交bug_openni2sdk](assets/提交bug_openni2sdk.png)

3. OrbbecViewer中无论是否勾选"镜像彩色流"，保存的彩色图都是镜像的，如下表。相对地，深度图、红外图没有这个bug。

   | OrbbecViewer中勾选"镜像彩色流"                             | OrbbecViewer中不勾选"镜像彩色流"                             |
   | ---------------------------------------------------------- | ------------------------------------------------------------ |
   | ![OrbbecViewer_勾选镜像](assets/OrbbecViewer_勾选镜像.png) | ![OrbbecViewer_不勾选镜像](assets/OrbbecViewer_不勾选镜像.png) |



## 摄像头和开发板性能限制影响开发的问题点描述

1. Astra Pro深度相机的点云精度（3mm @ 1m）远没有工业级点云相机（0.2mm @ 1m）高。基于Astra Pro深度相机，本项目的实际引导精度在5mm左右，可以满足工业现场下cm级的引导需求。如果深度相机的精度能更高，则不仅可以做轨迹引导，甚至可以做工件装配。
2. Zora P1开发板的内存只有2GB，普通使用够了，但如果编译大型的库，可能就不够用了。



## 建议

如果想让奥比中光的深度相机和开发板更多地出货，一个方法是找到大客户，另一个方法是抓住普通开发者。而抓住普通开发者最重要的是构建良好的生态。奥比中光创建3D视觉开发者社区、发布与深度相机高度适配的开发板都是在构建生态。但可能还不够，建议从下面两个层面继续努力构建生态：

1. 硬件方面。**提供更多的配套硬件**。可以参考树莓派，提供各种传感器、显示屏、移动电源等。不一定要官方自己做，也可与第三方合作。
2. 软件方面：**提供更多的例程文档和预编译包**。比如Zora P1是有板载的GPIO的，完全可以接各种传感器，但是官方并没有提供相应的示例文档，限制了普通开发者使用Zora P1的更多可能性。再比如，Zora P1是与深度相机结合使用的，而3D处理常用的库包括PCL、Open3d等，在armbian下都找不到预编译包，需要花费大量的时间精力自己编译，甚至可能由于板子内存太小等原因编译失败，建议官方提供常用3D处理库PCL、Open3D等的预编译库。

