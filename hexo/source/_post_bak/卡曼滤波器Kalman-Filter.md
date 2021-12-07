---
title: 卡曼滤波器Kalman Filter
typora-root-url: ../../source
date: 2021-07-25 14:12:38
tags: 时序预测
categories: 算法
sticky:

---

- # Intro

  最近在学习伟大的Kalman Filter，这篇笔记主要是对国外的两篇介绍Kalman Filter的博客的翻译和一些个人理解，博客链接：[Kalman Filter For Dummies](https://link.jianshu.com?t=http://bilgin.esme.org/BitsAndBytes/KalmanFilterforDummies)， [How a Kalman filter works](https://link.jianshu.com?t=http://www.bzarg.com/p/how-a-kalman-filter-works-in-pictures/)。两篇博客的侧重的层次面不太一样，第一篇讲的层次比较浅显但是缺失了一些细节，第二篇讲的很深入但是有些抽象，这篇笔记将两篇博客结合起来，希望可以由浅入深的理解Kalman Filter。

  # Kalman Filter的简介和应用

  首先先介绍一下Kalman Filter（以后简称kf）。kf是一种线性二次估计算法，由Rudolf Kalman于1960年提出，在随后的阿波罗计划中的导航系统就主要应用了kf算法。kf用对系统在时域的测量来预测一些未知的变量在下一个时间点的状态，测量的对象包括噪音干扰和其他的不确定因素。kf相比于其它的预测算法有更高的精准度。

  kf可以用于任何包含未知信息的动态系统，然后自适地预测系统下一步的状态。哪怕环境的干扰很多很复杂，kf一般也可以得出很准确的结果。kf在连续变化的系统有很好的效果。它的优点是在预测时并不需要记录很多系统过去的状态，只需要知道上一状态的预测结果的误差协方差矩阵就可以了，所以kf可以算得很快，可以用于实时的动态系统。

  # 一个kf的小例子

  首先我们创造一个场景来引入kf算法。假设我们有一个机器人可以在丛林中穿梭，机器人的控制算法需要知道机器人的位置和速度然后才可以导航机器人的运动。我们用一个状态Xk向量来表示：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-ca473aa1071295e4.png?imageMogr2/auto-orient/strip|imageView2/2/w/186/format/webp)

  这个例子为了简化所以只考虑两个状态变量，在现实中可能会有更多的变量，更多的变量就意味着更多的信息，这更有利于kf的计算精准度。

  同时，这个机器人还有一个GPS传感器来测量距离和速度。这个GPS的精度在10米左右，但这是远远不够的，因为如果在丛林里有一个悬崖的话10米的精度并不能保证机器人可以及时的避免。所以单单依靠GPS的数据来控制是不够精准的。而且传感器的也会被干扰，所以传感器的数据并不是完全可依靠的。

  机器人的运动可以分为主动因素和外界干扰两个方面。主动因素是控制程序给电机发送的运动指令，它可以是向前向后，但这并不能保证机器人一定会沿着指定的方向运动，因为还有外界干扰的存在。比如说机器人遇到障碍了，被风吹了等等。所以外界干扰也是不可被忽视的一部分。

  在这里就可以引入kf算法中的两个概念，主观预测和客观测量。我们的主观预测就是基于我们对机器人发出的命令，如果是向前5米，那我们对下一个时间点的的主观预测就是向前5米。这个预测当然是不准的，因为我们没有考虑到外界的干扰。客观测量就是我们通过GPS得到的数据，首先这个数据是间接的，因为传感器的得到的数据只是一些电平信号，我们还需要将它们对应转换到现实世界的数据。其次这个数据也是不可靠的，理由见上。

  我们对机器人下一个状态的最优预测只能基于主观预测和客观测量这两个信息，但是哪一个更可靠一点呢？我们加入权重来让这个预测结果更公平一点，这里就引入了我们最优预测的权重公式：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-401abb729ad86ae7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  其中Zk是我们得到的客观测量值，X（k－1）就是我们上一次做出的主观的预测，Xk就是我们最终的预测值，而K就是我们要通过kf找到的权重值，又称Kalman Gain。这样得到的估计相比于独立的主观估计和客观估计都要精确的多。

  但是注意，这个最优预测并不是用于实际的使用，实际中还是使用主观预测为估计，但是当到下一个时间点得到客观测量后，我们可以修正我们的估计，得到最优估计，这个估计是作为参数传递给下一次估计，这样一次次修正，我们的估计会更加精确。

  # 分布函数与协方差矩阵

  在我们的问题情景里，我们只有两个需要追踪的状态，位置和速度。我们可以假设位置和速度都服从于高斯分布，kf算法中，都假设要追踪的变量服从于高斯分布。一维高斯分布的分布函数为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-18b70a0e7f86af95.png?imageMogr2/auto-orient/strip|imageView2/2/w/764/format/webp)

  中间的μ是这个高斯分布的期望，σ是标准差，σ^2是方差，在这个分布里只有一个变量，这个函数的积分为1。现在我们把高斯分布扩展到二维，则分布函数为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-0298b51928e225f9.png?imageMogr2/auto-orient/strip|imageView2/2/w/406/format/webp)

  在这个图像中，X轴Y轴分别代表一个变量，Z轴是概率，所以中间最高点对应的X，Y为这两个变量的期望，整个凸起部分的体积积分为1。用2维的图来表示则为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-bcdd92266b698c37.png?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-9fc97efb8f02a542.png?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  颜色越白的地方概率越大，所以期望在中心点，周围黑色的部分概率为0。注意上图中所标注的方差的大小不是所标注的范围，此处有误解。

  在以上的分布的例子里，位置和速度的相关性不是很强，但是在现实中，位置一般是和速度有很大相关性的，比如以下的例子：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-1cf54162b32345aa.png?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  在这个例子中，速度与位置成正相关，当速度大的时候，往往位置也会大，速度小的时候，位置也会小。这样的相关性信息也是kf算法所需要的。多维变量的相关性的信息是用协方差矩阵进行储存的。协方差矩阵可以储存一个一维列向量的变量之间相互的相关性信息。在这个例子中一维的变量向量和协方差矩阵为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-526d6eca3e23c93a.png?imageMogr2/auto-orient/strip|imageView2/2/w/431/format/webp)

  自己和自己的相关性是1，所以Σpp和Σvv都是1。相关性与顺序无关，所以Σpv等于Σvp。所以协方差矩阵Pk是一个对称矩阵。

  # 主观预测与状态空间方程的构建

  首先要构建的是状态空间方程，所以变量为位置和速度。k是时间变量，我们的主观预测需要从（k－1）时的状态来预测k时的状态：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-4118364ea0e8f161.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  蓝色的区域是（k－1）时的速度与位置服从的分布，紫色的是k时速度与位置服从的分布。这个图假设不同的时间点速度与位置服从的分布不同，然而在大部分时候，速度与位置的分布是不随时间变化而变化的。

  在这里，我们做的是主观预测，我们的预测是给予一个经验公式，这个公式可以来自于控制的命令，或者其它的一些我们已知的信息。我们的预测需要将（k－1）的状态的分布转移到k的状态的分布，每一个（k－1）的点都可对应到K时的一个点。假设我们状态转移矩阵是Fk则：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-d0a5eeac79ed9da9.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  注意，一般情况下，这个Fk就是1，因为分布的关系不会随时间变化而变化。

  我们可以把这个state space 方程写下来：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-e16ad142b84429ba.png?imageMogr2/auto-orient/strip|imageView2/2/w/331/format/webp)

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-83ba43baacca96c6.png?imageMogr2/auto-orient/strip|imageView2/2/w/343/format/webp)

  Δt是k－1到k的时间变化，在这里假设机器人匀速运动。这样我们就得到了我们的初步的主观预测方程。我们可以从（k－1）的状态预测下一个时间点时位置和速度的值。同时根据协方差矩阵的性质：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-786742cef681a87d.png?imageMogr2/auto-orient/strip|imageView2/2/w/336/format/webp)

  我们可以得到一个完整的预测信息：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-ae17f1f75e01956b.png?imageMogr2/auto-orient/strip|imageView2/2/w/309/format/webp)

  但是这个状态方程只是一个很基础的，我们还需要考虑其他因素对系统的影响，比如说电机对机器人的控制，外界干扰等等。

  我们首先考虑电机对机器人的控制，如果加上电机的加速度，则空间状态方程为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-f458ed5fd1ca6a8f.png?imageMogr2/auto-orient/strip|imageView2/2/w/451/format/webp)

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-eb5ae2cf58fb6085.png?imageMogr2/auto-orient/strip|imageView2/2/w/406/format/webp)

  Bk就是控制矩阵，uk就是加速度向量。

  此外还要考虑噪声的影响，如果加上噪声，则每一个（k－1）时的点不会对应到K时的一个点，而是对应到一片区域，所以对应关系如下：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-42a35891a7ae826c.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  因为每一个点都对应到一片区域，所有的对应叠加在一起，得到的新的K的分布范围应该比以前要大，如下图所示：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-a0cbf225f18784ac.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  面积明显比上图要大，同时我们假设噪声是高斯白噪声，所以期望为0，所以加上噪声后K的期望不变。假设噪声所造成的那个小绿圈的速度与位置的协方差矩阵为Qk，则新的主观预测和协方差矩阵为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-207dd335f067c033.png?imageMogr2/auto-orient/strip|imageView2/2/w/347/format/webp)

  通过这个方程我们可以在X（k－1）时主观的预测出Xk时的状态。下一步就是得到客观测量的结果。

  # 客观测量

  我们的机器人上有GPS传感器可以得到位置和速度的信息，假设sensor1测的是速度，sensor2测的是位置：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-ba7bde56eb0afeee.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  但是注意，因为传感器得到的只是电平值，所以得到的分布函数的范围与现实中的速度和位置的范围不一样。我们需要用一个转移矩阵把现实世界的速度位置数据的map到传感器的电平数据单位，每一个点对应传感器坐标的一个点，然后在相同的单位下进一步计算权重Kalman gain。这里我们引入转移矩阵Hk：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-f82f0c7ee3c1dd30.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  对应的期望与协方差被转换为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-6752300f769af923.png?imageMogr2/auto-orient/strip|imageView2/2/w/335/format/webp)

  这里的μ expected就是我们的主观估计在传感器的单位下的值，Σ expected则是我们的主观估计的协方差矩阵在传感器单位的转换后的矩阵。

  当然，我们也不能忽略传感器的噪声影响，噪声会造成我们在把现实世界的值转换为传感器单位时一个点对应传感器坐标系的一片区域：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-77204d4f97f164ee.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  所以当传感器坐标系中每一个点都被这样的一个绿色的小椭圆代替，新的传感器坐标系下的数据分布将变为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-14b9d70d8326682b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  形状变成这样是因为我们假设的传感器噪音是这样的一个椭圆形，原来的紫色的小圆形被扩张成了这样的绿色椭圆。

  则绿色椭圆分布的期望（中心点）就是我们传感器读到的客观测量数据，但是包含误差。

  # 得出最优修正预测

  得到我们的主观预测在传感器坐标系的分布和客观测量得到的数据的分布后，就可以计算权重了。首先我们先将两个分布重叠在一起，得到如下分布图：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-17458063c76ab8ff.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  紫色的是主观估计，绿色的是客观测量。所以我们就有了两个概率：

  1，我们的主观估计是我们最优估计的概率

  2，客观测量是我们最优估计的概率

  在这里我们直接将两个概率分布函数相乘，得到一个新的分布。因为黑的地方概率为0，所以相乘后紫色和绿色的非重叠面积都变为0了。我们只得到中心重叠的区域为我们最优的预测的分布：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-079300d3b68259c1.png?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

  这个分布的期望就是我们对下一个时间点K的最优预测。这个预测的特点是，在这个点上，主观估计和客观测量的概率最大。

  一个一维更直观的图示为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-920a2983ebdce335.png?imageMogr2/auto-orient/strip|imageView2/2/w/589/format/webp)

  那个小蓝色分布的期望就是红色分布和绿色分布同时预测时的最佳预测值。

  # 计算权重Kalman Gain

  我们首先考虑只有一个变量时的情况。这时分布的波动情况用方差表示。

  因为我们是直接将两个分布函数相乘得到一个新的分布，所以用公式表达为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-3771ebeb7069ed12.png?imageMogr2/auto-orient/strip|imageView2/2/w/631/format/webp)

  如果两个分布都是服从高斯分布：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-887dd07dde191ec2.png?imageMogr2/auto-orient/strip|imageView2/2/w/406/format/webp)

  则相乘以后得到的新的分布的期望和方差为：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-5fd30537c356f9ce.png?imageMogr2/auto-orient/strip|imageView2/2/w/354/format/webp)

  我们可以在期望的计算公式中提出一个K，作为我们的权重Kalman Gain：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-bf917a818341f0f0.png?imageMogr2/auto-orient/strip|imageView2/2/w/350/format/webp)

  下面我们考虑多个变量的情况，这时数据的波动情况就不能用方差表示了，应该用协方差矩阵来表示，所以直接替换得到的新的分布和新的K：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-a4176932cdd6fcd7.png?imageMogr2/auto-orient/strip|imageView2/2/w/330/format/webp)

  这样，我们就得到多变量时的Kalman Gain。这个分布期望就是我们最终的预测。协方差矩阵就是我们要传给下一次预测的信息。

  # 推导公式

  我们的主观预测的分布是：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-72ffc1aa3319f70c.png?imageMogr2/auto-orient/strip|imageView2/2/w/391/format/webp)

  客观测量的分布是：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-b1f2ced7b66b5c62.png?imageMogr2/auto-orient/strip|imageView2/2/w/267/format/webp)

  所以代入刚才得到的最佳预测的分布公式得到：

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-4486502f17384b2a.png?imageMogr2/auto-orient/strip|imageView2/2/w/453/format/webp)

  K‘就是我们的最佳预测的权重Kalman Gain，Xk’就是我们的最佳预测，Pk‘就是我们要传给下一次预测的协方差矩阵。至此一次预测已经完成。

  # 迭代流程图

  

  ![img](https:////upload-images.jianshu.io/upload_images/1816530-34619539c3d01400.gif?imageMogr2/auto-orient/strip|imageView2/2/w/475/format/webp)

  我们首先给以个起始估计，然后在（k－1）时先做主观估计，用主观估计值作为我们的估计，然后等到K时，用得到客观测量进行矫正我们的估计，再把矫正后的新的值作为这次的最佳估计和新的协方差矩阵传递给下一次估计。

  Time Update是在（k－1）阶段，Measurement Update是在K阶段。等到K时我们才可以得到客观测量值，所以是用客观测量进行矫正之后的最佳估计值再用来下一次估计。但这个最佳估计并没有被用在实际中，之前实际用到的估计还是Time Update的估计值。

  # 简单的计算例子

  参见[A Simple Example](https://link.jianshu.com?t=http://bilgin.esme.org/BitsAndBytes/KalmanFilterforDummies#A Simple Example)

  ### A Simple Example

  Now let's try to estimate a scalar random constant, such as a "*voltage reading*" from a source. So let's assume that it has a constant value of **a**V (volts), but of course we some noisy readings above and below **a** volts. And we assume that the standard deviation of the measurement noise is 0.1 V.

  Now let's build our model:

  ![Kalman Filter - Example Equation 1](/images/%E5%8D%A1%E6%9B%BC%E6%BB%A4%E6%B3%A2%E5%99%A8Kalman-Filter/example_equation_1.gif)![Kalman Filter - Example Equation 2](/images/%E5%8D%A1%E6%9B%BC%E6%BB%A4%E6%B3%A2%E5%99%A8Kalman-Filter/example_equation_2.gif)

  > 这里A=1， uk=0，H=1：
  >
  > - Buk是控制因子，在bzag博客中举例为加速度变量，普通一维序列中不存在，所以这一项为0
  > - A是状态转移矩阵，如果假设前后时刻分布是一致的这里就1（一维的情况这个值是个常数而非矩阵，如果常量不取1，那就是线性增长的趋势，不符合）
  > - 第二个等式表示，客观测量值是估计值的线性组合以及噪声的结果。同样的，这里考虑没有线性特征的数据，所以H为1

  

  As I promised earlier, we reduced the equations to a very simple form.

  - Above all, we have a 1 dimensional signal problem, so every entity in our model is a numerical value, not a matrix.
  - We have no such control signal **u**k, and it's out of the game
  - As the signal is a constant value, the constant **A** is just 1, because we already know that the next value will be same as the previous one. We are lucky that we have a constant value in this example, but even if it were any other linear nature, again we could easily assume that the value **A** will be 1.
  - The value **H** = 1, because we know that the measurement is composed of the state value and some noise. You'll rarely encounter real life cases that **H** is different from 1.

  

  And finally, let's assume that we have the following measurement values:

  | **TIME** (ms) | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
  | ------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | **VALUE** (V) | 0.39 | 0.50 | 0.48 | 0.29 | 0.25 | 0.32 | 0.34 | 0.48 | 0.41 | 0.45 |