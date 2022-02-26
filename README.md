# Lesion-segmentation-of-head-neck-and-throat-cancer
基于Paddleseg的咽喉癌病灶分割，并且使用小工具本地部署。

# 一、项目背景介绍
喉癌和下咽癌是常见的头颈部恶性肿瘤，不仅会导致患者的发音、呼吸和吞咽功能失常，还会严重威胁患者生命。增强 CT 成像是临床评估喉癌和下咽癌的重要手段。然而，头颈癌被认为是放射肿瘤学领域中轮廓绘制最困难和最耗时的疾病部位之一。这项任务通常是人工完成的，是一项耗时且劳动密集型的任务。另一方面，最终的诊断结果容易受到不同医师之间的主观因素影响而存在差异。

近年来，虽然喉癌和下咽癌自动化分割算法的研究有了很大的发展，但仍面临一些有待解决的问题。因此，研究一种准确、实用的喉癌和下咽癌分割算法对于治疗方案的制定和放疗总成本的降低有重要意义。

本此项目使用UNet模型对3D增强CT图像进行窗宽窗位校正处理，再转存为2D图像后对其进行病灶分割，并使用PyQt5制作的图形化小工具进行本地部署。

# 二、数据介绍

此次选择的是2021 年（第14 届）中国大学生计算机设计大赛-人工智能应用-人工智能挑战赛-医学影像挑战赛提供的国赛数据集。该数据集为头颈部三维增强CT影像，所有数据均为 NiFTI 格式三维图像，不同患者使用数字格式编号。拥有90个头颈部三位增强CT影像与90个同等格式的标注数据

![](https://ai-studio-static-online.cdn.bcebos.com/b352df19314b42f78887ea949d050596f581cc4f4f724a3384b2066ad05882fe)


* 噪声多，病灶与正常组织对比度不高且边界模糊。所有病人的CT影像维度在XY轴方向上均为512×512，在Z轴方向上处于41-130之间。CT像素间距在XY轴方向上处于0.455mm-0.977mm之间，在Z轴方向上为3mm或5mm。CT值范围处于-1024.0-3071.0之间，噪声区域较多。

![](https://ai-studio-static-online.cdn.bcebos.com/fb1a712865244efd8c7e0b1fef7df350087b3e886fd14225a23dcd92a4b13bd7)


* 各病灶中心点不同

![](https://ai-studio-static-online.cdn.bcebos.com/e9c968a9c87f47d4b191bb92bef11bda00d44b79235b416b9d677a8214a94542)


* 正负样本比例失衡。病灶的总体积以及包含病灶的切片较少，90例数据总共7000+的切片，只有1000左右的切片包含病灶，病灶面积远远少于正常组织和整张图像面积。

![](https://ai-studio-static-online.cdn.bcebos.com/ee0cf7a51be4476d970d172defec597e275c20ab0b464444b0dbec1580c14ae7)


# **总体方案**

## 数据预处理
* 对3D增强CT图像进行窗宽窗位的校正并缩放了0到255，减少噪声；

* 该数据集的数据每个nii文件的图像横断面只有48~130张切片，与肝脏CT数据平均几百张切片相比，该数据更适合采用2D分割，所以需要将3D 图像转存为2D图像；

* 在转存为2D图像之前根据label数据和灰度值将nii图像进行阶段，否则负样本比例太高可能影响模型收敛；

* 预留出十个左右的样本作为测试集，预测之后根据3D Dice系数判断模型的优化程度。

## 模型搭建
* 使用水平反转、高斯噪声等数据增强策略进行数据扩充。因为病灶面积太小，使用缩放的时候尽量放大。
* 采用余弦退火学习率下降策略并配合AdamW进行训练能够较快地收敛。

## 图形化预测小工具使用
这个就得去看[吖查小哥哥](https://aistudio.baidu.com/aistudio/personalcenter/thirdview/181096)的[项目](https://aistudio.baidu.com/aistudio/projectdetail/2574999)啦，我就不多做解释了。

# 二、数据预处理

### 窗宽窗位校正

![](https://ai-studio-static-online.cdn.bcebos.com/2cca0eb9615e45fbb63a0e1fc7b42288fd18482905ab493f8da81e73d2b5cf17)


## 2.2 图像截断
根据标注数据和灰度值阈值进行截断，去掉一些多余切片

再次之前肯定得进行窗宽窗位的校正并且缩放到0-255，否则图像将是黑抹哒区的一片（当初进行归一化的图像分割出来的结果比没有进行归一化的图像分数更低，这就挺魔性的）

![](https://ai-studio-static-online.cdn.bcebos.com/6c3a4d32a8c146d1b7efb5502097e88c1b069b95c44e4055bbd4fb5c03a8b388)


## 2.3 将截断后的3D 数据根据Z轴切割转存为2D图像

![](https://ai-studio-static-online.cdn.bcebos.com/f4143047517d4f62b1bf5f150703e13b4296fd03430d41af8fc2d4284520dcc8)


# 三、模型介绍
![](https://ai-studio-static-online.cdn.bcebos.com/507ad8a1546c436ca506c9b91b2019d664fa7a88735c45048b70558989090cde)

细节方面请转到[【AI达人创造营第二期】作业四：头颈部咽喉癌病灶分割](https://aistudio.baidu.com/aistudio/projectdetail/3527063)
