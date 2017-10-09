---
 title: TensorFlow 初试
 date: 2017-01-19 23:55:43
 tags:  TensorFlow
 categories: 机器学习
---

## 安装

1. `pip install tensorflow`

2. 如果安装GPU版本的，需要安装 Cuda Toolkit 8.0 和 cuDNN v5.1
   `pip install tensorflow-gpu`
   并且设置环境变量 `LD_LIBRARY_PATH`和`CUDA_HOME`。

3. 运行 demo model

   `python -m tensorflow.models.image.mnist.convolutional`