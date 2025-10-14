---
layout: post
title: pytorch与dataloader
author: huangstomach
date: 2022-09-21 14:44:37 +0800
tag: DeepLearning
---

> 记录一下最近进行深度学习开发所使用的一些简单的操作

## Pytorch

[PyTorch](https://pytorch.org/)是一个开源的深度学习框架，它为研究工作提供了灵活和模块化，并具有生产部署所需的稳定性和支持。`PyTorch`为张量计算（如`NumPy`）等高级功能提供了一个`Python包`，具有强大的GPU加速功能，并提供`TorchScript`，以便在急切模式和图形模式之间轻松转换。随着`PyTorch`的最新发布，该框架提供了基于图形的执行、分布式训练、移动部署和量化。

虽然是由Meta的人工智能团队开发，但有时候Google团队居然也会使用。

## Dataset

在进行深度学习训练时，我们一般来说都会需要一个训练集，让我们的算法与模型来从其中学习规律（模式），从而达到基于学得的规律来进行决策的能力。

在使用的深度学习框架时，往往就需要使用它们对应的数据集类，来进行数据的读取和处理。

当然也可以不用，但是不使用的话在出现大规模计算等情况下自己实现一些方法太过复杂也没有必要，往往还是选择对应的框架提供的数据基类来进行实现。

针对`Pytorch`来说，初始化一个基础的数据集类非常简单，以映射数据集为例：

``` python
import torch
from torch.utils.data import Dataset

class MyDataset(Dataset):
    def __init__(self, train = True):
        self.device = device
        self.train = train
        
        self.feature_x1 = torch.randn(100, 1)
        self.feature_x2 = torch.randn(100, 1)
        self.y = torch.randn(100, 1)

    def __getitem__(self, index):
        return (
            self.feature_x1[index], 
            self.feature_x2[index], 
            self.y[index]
        )

    def __len__(self):
        return self.y.size(dim=0)

    def _check_exists(self):
        pass
```

只需要简单的继承`Dataset`类，然后实现`__init__`、`__getitem__`、`__len__`三个方法即可。

其中`__len__`方法需要返回数据集的数据条目长度（样本规模），`__getitem__`方法需要返回对应索引的数据。

代码中简单实现了一个具有两种特征的数据集，每个样本都有两个特征和一个标签，`__getitem__`方法返回的是一个元组，包含了对应索引的特征和标签。

对于比较小的数据集，可以一股脑的把数据丢入模型中，直接进行梯度下降计算。不过有些数据集（比如计算机视觉）非常的大，可能无法一次性加载到内存中，所以可以考虑只在`__init__`时存储索引，而在`__getitem__`时再从硬盘读取，这样虽然比较慢但是可以节省内存。

但有时候，我们需要使用GPU来进行训练，虽然整个数据集合在内存中可以存储，但传递到GPU中又有些勉强，这时候可以考虑使用`Dataloader`来进行数据的读取。

## Dataloader

``` python
from torch.utils.data import DataLoader

train = MyDataset()
trainLoader = DataLoader(train, batch_size=128, shuffle=True)

for epoch in range(1000):
    for x1, x2, y in trainLoader:
        pass
```

如代码所示，简易的DataLoader可以只传递三个参数，分别是数据集、`batch_size`和`shuffle`。

其中`batch_size`来决定每次在GPU中进行张量计算的数据量，`shuffle`来决定是否在每个epoch开始时对数据进行打乱。

这样在模型进行反向传播时就转化为了小批量梯度下降，虽然达到最优解的路程会比较坎坷，但是可以有效的减少内存占用，加快训练速度。

## Tips

有时候我们的数据集合比较简单，但是计算机性能非常强大，有时候却反而会出现训练速度非常的慢，对应计算单元的占用也非常的低。这时候有一种可能是进行计算推演的时间远远小于将数据集转移到张量计算上的时间，可以考虑进一步增加`batch_size`来减小到GPU之间的数据交换次数。

