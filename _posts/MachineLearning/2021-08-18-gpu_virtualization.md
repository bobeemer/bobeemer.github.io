---

layout: post
title: 提高gpu 利用率
category: 架构
tags: MachineLearning
keywords:  gpu

---

## 简介
* TOC
{:toc}


## gpu 共享

2. 支持共享gpu，按卡和按显存调度共存（主要针对推理和开发集群）。通过共享gpu 的方式提高资源利用率，将多应用容器部署在一张 GPU 卡上。 Kubernetes对于GPU这类扩展资源的定义仅仅支持整数粒度的加加减减， it's impossible for a user to ask for 0.5 GPU in a Kubernetes cluster。需要支持 以显存为调度标尺，按显存和按卡调度的方式可以在集群内并存，但是同一个节点内是互斥的，不支持二者并存；要么是按卡数目，要么是按显存分配。可以使用 [Volcano GPU共享特性设计和使用](https://mp.weixin.qq.com/s/byVNvnm_NiMuwiRwxZ_gpA) 和 [AliyunContainerService/gpushare-scheduler-extender](https://github.com/AliyunContainerService/gpushare-scheduler-extender) 等开源实现
3. 支持gpu隔离， 减少共享gpu 带来的pod 间干扰。多pod共享gpu卡，容易导致使用不均，比如两个pod调度到一个卡，其中一个pod有可能占用过多mem，导致另外一个pod oom，需要进行显存和算力隔离，[怎样节省 2/3 的 GPU？爱奇艺 vGPU 的探索与实践](https://www.infoq.cn/article/r6ffgqdvozfv8c5zc3et)基于 CUDA API 截获方式实现显存及算力隔离和分配，并基于开源项目 aliyun-gpushare scheduler实现 K8S 上对虚拟 GPU 的调度和分配，实现了多应用容器部署在一张 GPU 卡的目标。
1. 显存隔离，拦截特定的API 将其读取的显存的总量 改为 给容器设定的显存上限值
2. 算力隔离，CUDA 有三层逻辑层次，分别为 grid（对应整个显卡），block（对应SM），和 thread（对应SP）。SM 之间可以认为是没有交互的（不绝对），既然一些模型无法完全利用 GPU 的全部算力，那么何不削减其占用的 SM 个数，使得空闲下来的 SM 可以为其他 GPU 程序所用？

[深度剖析：针对深度学习的GPU共享](https://cloud.tencent.com/developer/article/1757129)





## 虚拟化（未完成）

[GPU虚拟化，算力隔离，和qGPU](https://cloud.tencent.com/developer/article/1831090)

[浅谈GPU虚拟化和分布式深度学习框架的异同](https://zhuanlan.zhihu.com/p/390493981)
1. GPU虚拟化的主要出发点是，有些计算任务的颗粒度比较小，单个任务无法占满整颗GPU，如果为每一个这样的任务单独分配一颗GPU并让这个任务独占，就会造成浪费。因此，人们希望可以让多个细粒度的任务共享一颗物理的GPU，同时，又不想让这些任务因共享资源互相干扰，也就是希望实现隔离。因此，GPU 虚拟化要实现“一分多”，也就是把一颗物理的GPU分成互相隔离的几个更小的虚拟GPU，也就是vGPU，每颗vGPU独立运行一个任务，从而实现多个任务共享一颗物理的GPU。
2. GPU虚拟化在推理任务中有需求，这是因为推理任务的负载取决于服务请求的多寡，在服务请求的低谷，推理需要的计算时有时无，多个不同的推理任务通过GPU虚拟化共享资源说的通。在模型调试环节或教学环境，GPU虚拟化也有需求，算法工程师或学生每改一次代码就启动一次任务，这个任务或者因为错误很快就停止，或者被工程师杀死，不会持续的需要资源。
3. 一般来说，正式的深度学习训练任务计算量都比较大，唯恐资源不够，而不是发愁资源多余，因此GPU虚拟化在训练过程中不太需要。在正式训练中如果出现 GPU 利用率不高的时候，采取的措施往往是对训练进程进行 profiling，找出 GPU 处于 idle 的原因，比如 io、通信、cpu 计算，而不是切分 GPU。



[开源GPU显存虚拟化项目，你的2080Ti还能救一下](https://zhuanlan.zhihu.com/p/391539554)

## 弹性训练

具体到工程上是各个 训练组件支持 动态地调整参与训练的实例数量。 worker 加入/移除训练后，训练任务会不中断地继续训练。

1. Horovod / mpi-operator[云原生的弹性 AI 训练系列之一：基于 AllReduce 的弹性分布式训练实践](https://mp.weixin.qq.com/s/X4VDynLfKdVp-tyciQccyQ)
2. pytorch [云原生的弹性 AI 训练系列之二：PyTorch 1.9.0 弹性分布式训练的设计与实现](https://mp.weixin.qq.com/s/hlOYLKSHFDZWN21AsUn6bg)
3. [云原生的弹性 AI 训练系列之三：借助弹性伸缩的 Jupyter Notebook，大幅提高 GPU 利用率](https://mp.weixin.qq.com/s/Hms33MbcSB2DERfQE2TcEg) 未读


实现弹性训练需要注意的问题
1. 弹性训练需要一种机制来解决节点/训练进程间相互发现的问题。Horovod 将这一问题交给用户来解决，Horovod 定期执行用户定义的逻辑来发现目前的节点。PyTorch 通过第三方的分布式一致性中间件 etcd 等来实现高可用的节点发现。
2. 要实现弹性训练还需要捕获训练失效。Horovod 和 PyTorch 都通过一个后台进程（Horovod 中是 Driver，PyTorch 中是每个节点的 Local Elastic Agent）来实现这一逻辑。当进程 crash，或在梯度通信中遇到问题时，后台进程会捕获到失效并且重新进行节点发现，然后重启训练。
3. 训练时的数据切分的逻辑和学习率/ batch size 的设置也要对应进行修改。由于参与训练的进程会动态的增减，因此可能需要根据新的训练进程的规模来重新设置学习率和数据分配的逻辑，避免影响模型收敛。