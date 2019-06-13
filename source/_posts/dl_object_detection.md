title: 目标检测 object detection
date: 2019/06/11
categories:
- Machine Learning
tags:
- Model
---


## 问题分析

输出：给出是否存在object的概率$p_{c}$，object的中心点$b_{x},b_{y}$和宽高$b_{w},b_{h}$，以及object属于哪个具体类别的指示$c_{1},c_{2},c_{3},...$

$(p_{c},b_{x},b_{y},b_{w},b_{h},c_{1},c_{2},c_{3},...)$


## 问题处理

- sliding windows
  - 训练：closely cropped data；检测：sliding windows with different sizes
  - sliding windows计算效率较低
  - 使用卷积替代FC，使得图像整体输入处理，即可得到sliding windows结果（提高计算效率）
- YOLO
  - 输入打格子，不做滑动，每个格子粒度构建label训练、检测
  - $b_{w},b_{h}$可能大于1（跨格子）
  - 非最大值抑制 non-max suppression：高IoU的一系列格子，只选$p_{c}$最大的
  - 多object在同一个grid：anchor box预定义bounding box形状，来区分不同的object
    - 使用前：各object <-> grid cell
    - 使用后：各object <-> (grid cell, bounding box)


## 评价指标

- 交并比 intersection over union，IoU









