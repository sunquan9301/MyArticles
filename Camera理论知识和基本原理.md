@[TOC](Camera理论知识和基本原理)
# 1. 前言
<font size="3">本篇文章为Camera系列文章的第一篇，主要阐述Camera摄像头的**基础理论知识**，解决Camera硬件或Camera软件开发的一些困惑。该系列文章主要围绕Android操作系统进行，并涉及Android系统上Camera的协议、实现和应用。</font>

<font color="red">转载请注明出处。可私信、也可邮件、联系方式见文章末尾。</font>

# 2. Basic Concepts
如下表[^1]：
[^1]: [Wikipedia](https://en.wikipedia.org/wiki/Main_Page)

|  术语 | 解释 | 备注 |
|:--------:|:------:|--|
| Camera | an optical instrument that captures a visual image. | 光学仪器捕获视觉形象显示在感光表面 |
| Image sensor |detects and conveys information used to make an image.  | 两种主流传感器为**CCD**和**CMOS**|
| CCD |charge-coupled device | 电荷耦合|
| CMOS |Complementary Metal-Oxide Semiconductor| 互补性氧化金属半导体|
|  pixel |像素| 相机传感器上的最小感光单位|
| IR Filter |红外滤光片| 人眼看不到红外光，但sensor可，需将红外光滤掉使图像更接近人眼看到效果。|
|  Lens | 镜头 | 分为塑胶透镜(plastic)或玻璃透镜(glass) |
|  DSP | Digital signal processor, | 就是把Raw RGB格式转换成RGB格式或者是YUV格式 |
|  ISP | Image signal processor, | 把 sensor 采集到的原始数据转换为显示支持 的格式 |
|  3A算法 | AF、AE、AWB | 自动对焦、自动曝光、自动白平衡  |
|  A/D转换 |  analog signals to digital format |   |
# 3. 总体流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210706231355543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
<font size="3">
涉及如下几个步骤
 1. 光线通过镜头Lens进入摄像头内部。
 2. 图像传感器转换光信息为电信息
 3. 原始的RAW Data经过DSP转换成RGB/YUV数据
 4. RGB/YUV通过应用层展现
</font>
# 4. 摄像头
![摄像头结构图](https://img-blog.csdnimg.cn/20210706231733611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
<font size="3">摄像头的通用硬件结构如上图所示。利用透镜折射原理，景物光线透过镜头在聚焦平面上形成清晰的像，然后通过感光材料CMOS或CCD记录影像，并通过电路转换为电信号。[^2]</font>

[^2]: [摄像头工作原理](https://blog.csdn.net/ysum6846/article/details/54380169)

<font size="3">Lens一般是有几片透镜组成，按材质可分为

 - Plastic ：塑胶透镜
 - Glass ：玻璃透镜
</font>

<font size="3">
Glass比Plastic贵。在透光率和感光性等光学指标上也比Plastic要好。手机考虑到成本，一般使用的是Plastic。
</font>

&nbsp;

<font size="3">
结合以上2种结构，摄像头采用的镜头结构有：1P、2P、1G1P、1G2P、2G2P、2G3P、4G、5G等。透镜越多，成本越高，相对成像效果会更出色。
</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210706232725362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
# 5. 传感器 Sensor 

<font size="3">

 - 负责将Lens的光信号转换为电信号，再进过内部AD转换为数字信号。 
 - 由于 Sensor 的每个 pixel 只能感光 R 光或 B 光或 G 光，因此每个像素此时存储的是单色的，称之为 RAW DATA 数据。
 - 想将每个像素的 RAW DATA 数据还原成三基色，就需要 ISP 来处理。
 - 目前常用Sensor主要有**CCD**和**CMOS**：
 
</font>

## 5.1 CCD（Charge Coupled Device) 电荷耦合器件传感器
<font size="3">

 - 高感光度半导体材料制成，由许多感光单位组成，通常以百万像素为单位。
 - 当CCD表面受到光线照射时，每个感光单位会将电荷反映在组件上，所有的感光单位所产生的信号加在一起，就构成了一幅完整的画面。
</font>

## 5.2 CMOS（Complementary Metal-Oxide Semiconductor）互补性氧化金属半导体
 - 硅和锗做成的半导体，使其在CMOS上共存着带N(-)和P(+)级的半导体，这两个互补效应所产生的电流可以被处理芯片记录并解读成影像。
 - CMOS传感器主要以美国、韩国和中国台湾为主导，主要生产厂家是美国的OmnVison、Agilent、Micron，中国台湾的锐像、原相、泰视等，韩国的三星、现代。
## 5.3 CCD VS CMOS
#### 5.3.1 CCD特点
<font size="3">
技术成熟、成像质量高、灵敏度高，噪声低，响应速度快，有自扫描功能，图像畸变小，无残像、应用超大规模集成电路工艺技术生产，像素集成度高，尺寸精确
</font>

#### 5.3.2 CMOS特点
<font size="3">
读取信息的方式简单、输出信息速率快、耗电省（仅为CCD芯片的1/10左右）、体积小、重量轻、集成度高、价格低等特点。
</font>

#### 5.3.3 特点分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707134553393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
<font size="3">
如图-CCD和CMOS的感光二极管排列。
 - CCD阵列，是在仅有一条总线后加A/D转换(紫色箭头，见2. Basic Concept)。在时钟信号同步下，一步步移位读对应二极管的电平值。
 - CMOS阵列，是在每个感光二极管旁都加入了A/D转换。**是主动式输出采集的数据信息**，而**CCD**是在同步电路控制下**被动式的输出采集的数据。**

因此我们详细在性能，耗电，画质等方面做初步比较

</font>

**性能**
-  CCD需在同步时钟控制下以行为单位一位一位输出信息，速度当然慢不适合快速连拍，保存图像速度慢。
-  CMOS阵列有坐标嘛，传感器采集光信号的同时就可以取出电信号，保存图像速度快。

**耗电**
-   CCD耗电大，在同步信号控制下一位位读取，需要时钟控制电源和三组电源供电。
-  CMOS耗电小。CMOS传感器经光电转换后直接产生电流或电压信号，信号读取十分简单，而且感光二极管所需的电压，直接由晶体放大输出，所以需要施加在源极的电平很小

**画质**

- CMOS是主动式输出数据，阵列上每个点都要经过两条传输总线，路程长，传输时的噪声引入多。
-  CMOS阵列的每个二极管旁边都有A/D，光电传感元件与电路之间距离很近，相互之间的光电磁干扰较为严重，放大的同时可能带入的噪声也大。 
-  CMOS因为二极管旁带有A/D电路，同样尺寸的sensor，二极管能受光线面积小，一部分光线被浪费了，受光弱于CCD的感光二极管，带入的一点小噪声就会被放大。 

**发展**
<font size="3">
CCD传感器制作技术起步较早，技术比较成熟。近年来，CMOS技术发展一日千里，在中小尺寸传感器上，CMOS和CCD的画质区别已经很小。
</font>
## 5.4 传感器尺寸和画质的关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707142153679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
**传感器尺寸大小对于画质的影响，就是采集光线数据的正确性和完整性的不同。**
1. 传感器面积越大，感光阵列面积就越大，相邻感光电路的距离就越大，加电时产生的电磁干扰就越小。 
2. 传感器面积越大，感光阵列面积就越大，对应单个像素的透镜就能做的越大，聚集到的光线就越多，感光二极管受光后产生的输出电平就越高。带来更高的信噪比，转换后的信息处理正确率就越高。 

<font color="red" size="3">传感器自己就总结到这里，如果感兴趣可查看注脚或搜集更多资料，深入探究。[^3] [^4]</font>

[^3]: [camera基本知识](https://blog.csdn.net/u012900947/article/details/81673639)
[^4]: [详解CCD和CMOS的区别及主要硬件技术指标](https://ee.ofweek.com/2017-01/ART-8500-2806-30086986.html)
# 6. ISP and DSP
<font  size="3">由于本人专业知识有限，不深入进行总结分析，感兴趣可参阅数字信号处理等相关书籍[^3] [^5] [^6]</font>
[^5]:[ISP流程概述](https://blog.csdn.net/u012900947/article/details/81624878)
[^6]: [ISPfuncs](https://blog.csdn.net/g_salamander/article/details/8086835)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707152822635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
<font size="3">
 - ISP 处理Image Sensor（图像传感器）的输出数据，如AEC（自动曝光控制）、AGC（自动增益控制）、AWB（自动白平衡）、色彩校正、Gamma校正、祛除坏点、Auto Black Level、Auto White Level 等等。
 - DSP功能较多，它可以做些拍照以及回显（JPEG的编解码）、录像以及回放（Video 的编解码）、H.264的编解码、还有很多其他方面的处理，总之是处理数字信号了。
 - 可以认为ISP是一类特殊的处理图像信号的DSP。
 - 数字信号处理芯片，感兴趣的可参阅[^5]
</font>

# 7.图像
## 7.1 像素
<font size="3">一部手机屏幕的分辨率是1280×720，说明水平方向有720个像素点，垂直方向有1280个像素点，有1280×720个像素点（分辨率）。每个像素点都由三个子像素点。当要显示某篇文字或者某幅图像时，就会把这幅图像的每一个像素点的RGB通道分别对应的屏幕位置上的子像素点绘制到屏幕上，从而显示整个图像[^7]。</font>
[^7]:[音视频开发进阶指南](http://xcodep.oss-cn-beijing.aliyuncs.com/2020/09/20200912093929907.pdf)
## 7.2 RGB表示方式
<font size="3">通用图像数据格式，理论上任何颜色都可以用红绿蓝三种基本颜色混合而成。有RGB888、ARGB8888、RGB565等格式(Android)
- RGB565：共16比特，占2个字节。G站6位是因为人的眼睛对绿光最为敏感，可表示颜色数为2^16 = 65536色。
- RGB888：共24比特，占3个字节，可表示颜色数为2^24 = 1677W。
- ARGB8888：共32比特，占4个字节，可表示颜色数为2^32。
</font>
## 7.3 YUV表示方式

对于视频帧的裸数据表示，更多的是YUV数据格式，主要应用于优化彩色视频信号的传输，使其向后兼容老式黑白电视。相比RGB它最大的优点在于只需要占用极少的频宽（RGB要求三个独立的视频信号同时传输）。[^8]
[^8]: [一文理解 YUV](https://zhuanlan.zhihu.com/p/75735751)
- Y：Luma 明亮度，也称灰阶值，以前的黑白电视，是只有Y信号而没有UV信号。
- UV：Chroma 色度，色彩及饱和度，用于指定像素的颜色。 “色度”定义了颜色的两个方面——色调(Cr)与饱和度(Cb)
- Cr反映了RGB输入信号红色部分与RGB信号亮度值之间的差异
- Cb反映的则是RGB输入信号蓝色部分与RGB信号亮度值之间的差异。

## 7.4 YUV采样格式
主流有如下三种
- YUV 4:4:4 采样
- YUV 4:2:2 采样
- YUV 4:2:0 采样

#### 7.4.1 YUV 4:4:4

<font size="3">YUV 4:4:4 表示 Y、U、V 三分量采样率相同，即每个像素的三分量信息完整，都是 8bit，每个像素占用 3 个字节。</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707161241839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)

> 四个像素为： [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]
采样的码流为： Y0 U0 V0 Y1 U1 V1 Y2 U2 V2 Y3 U3 V3
映射出的像素点为：[Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]

#### 7.4.2 YUV 4:2:2
<font size="3">YUV 4:2:2 表示 UV 分量的采样率是 Y 分量的一半。</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707161337931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)

> 四个像素为： [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]
采样的码流为： Y0 U0 Y1 V1 Y2 U2 Y3 U3
映射出的像素点为：[Y0 U0 V1]、[Y1 U0 V1]、[Y2 U2 V3]、[Y3 U2 V3]

<font size="3">其中，每采样一个像素点，都会采样其 Y 分量，而 U、V 分量都会间隔采集一个，映射为像素点时，第一个像素点和第二个像素点共用了 U0、V1 分量，以此类推。从而节省了图像空间。</font>

<font size="3">比如一张 1920 * 1280 大小的图片，采用 YUV 4:2:2 采样时的大小为：</font>

> (1920 * 1280 * 8 + 1920 * 1280 * 0.5 * 8 * 2 ) / 8 / 1024 / 1024 = 4.68M

#### 7.4.3 YUV 4:2:0
<font size="3">YUV 4:2:0 并不意味着不采样 V 分量。它指的是对每条扫描线来说，只有一种色度分量以 2:1 的采样率存储，相邻的扫描行存储不同的色度分量。也就是说，如果第一行是 4:2:0，下一行就是 4:0:2，在下一行就是 4:2:0，以此类推。</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707162656516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)

> 图像像素为：
[Y0 U0 V0]、[Y1 U1 V1]、 [Y2 U2 V2]、 [Y3 U3 V3]
[Y5 U5 V5]、[Y6 U6 V6]、 [Y7 U7 V7] 、[Y8 U8 V8]
​
采样的码流为：
Y0 U0 Y1 Y2 U2 Y3 
Y5 V5 Y6 Y7 V7 Y8
​
映射出的像素点为：
[Y0 U0 V5]、[Y1 U0 V5]、[Y2 U2 V7]、[Y3 U2 V7]
[Y5 U0 V5]、[Y6 U0 V5]、[Y7 U2 V7]、[Y8 U2 V7]

<font size="3">一张 1920 * 1280 大小的图片，采用 YUV 4:2:2 采样时的大小为：</font>

> (1920 * 1280 * 8 + 1920 * 1280 * 0.25 * 8 * 2 ) / 8 / 1024 / 1024 = 3.51M

<font size="3">相比较RGB节省了一半的空间。</font>

## 7.5 YUV存储格式
<font size="3">YUV 数据有两种存储格式：平面格式（planar format）和打包格式（packed format）</font>

- planar format：先连续存储所有像素点的 Y，紧接着存储所有像素点的 U，随后是所有像素点的 V。
- packed format：每个像素点的 Y、U、V 是连续交错存储的。

<font size="3">不同的采样方式和存储格式，就会产生多种 YUV 存储方式，这里介绍基于YUV420 的存储方式。</font>

**YUV420P 和 YUV420SP**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707163824400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Njb3R0X1M=,size_16,color_FFFFFF,t_70)
<font size="3">YUV420P 是基于 planar 平面模式进行存储，先存储所有的 Y 分量，然后存储所有的 U 分量或者 V 分量。</font>

<font size="3">同样，YUV420SP 也是基于 planar 平面模式存储，与 YUV420P 的区别在于它的 U、V 分量是按照 UV 或者 VU 交替顺序进行存储。</font>

**YU12 和 YU21**

<font size="3">YU12 和 YV12 格式都属于 YUV 420P 类型，即先存储 Y 分量，再存储 U、V 分量，区别在于：YU12 是先 Y 再 U 后 V，而 YV12 是先 Y 再 V 后 U 。</font>

**NV21 和 NV21**
<font size="3">NV12 和 NV21 格式都属于 YUV420SP 类型。它也是先存储了 Y 分量，但接下来并不是再存储所有的 U 或者 V 分量，而是把 UV 分量交替连续存储。</font>

- <font size="3">NV12 是 IOS 中有的模式，它的存储顺序是先存 Y 分量，再 UV 进行交替存储。</font>

- <font size="3">NV21 是 安卓 中有的模式，它的存储顺序是先存 Y 分量，在 VU 交替存储。</font>

## 7.5 RGB和YUV的相互转化
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707164131556.png)
# 8. 3A算法
## 8.1 AF 自动对焦
<font size="3">目的是获得清晰度更高得图像。方法分两类，一类是传统的聚焦方法，一种是基于数字图像处理方式的图像聚焦方法。传统的方式中，自动聚焦通过红外线或者超生波测距的方式来实现。这种方式需要安装发射机和接收机，增加了摄像机的成本，而且超声波对于玻璃后面的被摄物体不能很好的自动聚焦。这一类聚焦方式在某些场合受到了限制。因此在日趋集成化、微型化、低成本的应用中，基于数字图像处理的自动聚焦方法更具有优势。</font>

<font size="3">根据镜头成像分析，镜头的光学传递函数可以近似为高斯函数，它的作用等效为一个低通滤波器。离焦量越大，光学传递函数的截止频率越低。从频域上看，离焦量增大，对图像高频能量造成损失，使得图像的细节逐渐模糊。从空域上看，离焦量增大，点光源成像的光强分布函数越分散，可分辨的成像间距越大，图像相邻像素互相重叠，图像细节损失严重。因此图像清晰度评价函数时建立在图像边缘高频能量上的。</font>

<font size="3">数字处理方法中，自动聚焦的关键在于构造图像的清晰度评价函数。己经提出的图像清晰度评价函数苞括灰度方差、梯度能量、嫡函数和一些频域函数法。图像清晰度评价函数必须具有良好的单峰性和尖锐性，而且要计算量适度，从而可以快速的实现精准对焦。</font>

## 8.2 AE 自动曝光
<font size="3">曝光是用来计算从景物到达相机的光通量大小的物理量。图像传感器只有获得正确的曝光，才能得到高质量的照片。曝光过度，图像看起来太亮曝光不足，则图像看起来太暗。到达传感器的光通量的大小主要由两方面因素决定：曝光时间的长短以及光圈的大小。</font>

<font size="3">利用光圈进行自动曝光，主要根据所拍摄的场景来控制光圈大小，使得进光量维持在一定范围内。通过光圈进行曝光控制的成本比较高。现在市场所见的中低端摄像头采用的主流技术通过调整曝光时间来实现自动曝光。</font>

<font size="3">目前自动曝光控制算法方法有两种，一种是使用参照亮度值，将图像均匀分成许多的子图像，每一块子图像的亮度被用来设置参照亮度值，这个亮度值可以通过设置快门的速度来获得。另外一种方法是，通过研究不同光照条件下的亮度与曝光值之间的关系来进行曝光控制。这两种方法都是研究了大量的图像例子和许多不同的光照条件。而且均需要在不同的光照条件下所采集的图像数据库。实际中自动曝光研究需要解决好以下几个问题，首先是判定图像是否需要自动曝光，其次是自动曝光时，如何调整光电转换后数字信号来找出自动曝光能力补偿函数，最后就是调整到什么程度最为合适。</font>

## 8.3 AWB 自动白平衡
<font size="3">白平衡的基本原理是在任意环境下， 把白色物体还原成白色物体， 也就是通过找到图像中的白块， 然后调整R/G/B 的比例， 如下关系：
R’= R * R_Gain
G’ = G * G_Gain
B’ = B * B_Gain
R’ = G’= B’
AWB 算法通常包括的步骤如下：
(1)色温统计： 根据图像统计出色温；
(2)计算通道增益： 计算出R 和B 通道的增益；
(3)进行偏色的矫正： 根据给出的增益， 算出偏色图像的矫正。
</font>

# 9 联系方式
<font size="3">个人职业：阿里巴巴无线开发专家 可内推各大公司岗位
WeChat：sunquan97 添加请注明CSDN+原因
email：sunquan9301@163.com
