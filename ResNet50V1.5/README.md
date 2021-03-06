<!-- omit in toc -->
# Paddle ResNet50V1.5 性能测试

此处给出了Paddle ResNet50V1.5的详细测试报告，包括执行环境、Paddle版本、环境搭建方法、复现脚本、测试结果和测试日志。

同时，给出了在同等执行环境下，业内几个知名框架在ResNet50V1.5模型下的性能数据，进行对比。

其他深度学习框架的 ResNet50V1.5 训练性能数据测试流程，请参考：[OtherReports](./OtherReports)。

<!-- omit in toc -->
## 目录
- [一、测试说明](#一测试说明)
- [二、环境介绍](#二环境介绍)
  - [1.物理机环境](#1物理机环境)
  - [2.Docker 镜像](#2docker-镜像)
- [三、环境搭建](#三环境搭建)
  - [1.单机（单卡、8卡）环境搭建](#1单机单卡8卡环境搭建)
  - [2.多机（32卡）环境搭建](#2多机32卡环境搭建)
- [四、测试步骤](#四测试步骤)
  - [1.单机（单卡、8卡）测试](#1单机单卡8卡测试)
  - [2.多机（32卡）测试](#2多机32卡测试)
- [五、测试结果](#五测试结果)
  - [1.Paddle训练性能](#1paddle训练性能)
  - [2.与业内其它框架对比](#2与业内其它框架对比)
- [六、日志数据](#六日志数据)

## 一、测试说明

我们统一使用 **吞吐能力** 作为衡量性能的数据指标。**吞吐能力** 是业界公认的、最主流的框架性能考核指标，它直接体现了框架训练的速度。

Resnet50V1.5 作为计算机视觉领域极具代表性的模型。在测试性能时，我们以 **单位时间内能够完成训练的图片数量（images/sec）** 作为训练期间的吞吐性能。在其它框架中，默认也均采用相同的计算方式。

测试中，我们选择如下3个维度，测试吞吐性能：

- **卡数**

   本次测试关注1卡、8卡、32卡情况下，模型的训练吞吐性能。选择的物理机是单机8卡配置。
   因此，1卡、8卡测试在单机下完成。32卡在4台机器下完成。

- **FP32/AMP**

   FP32 和 AMP 是业界框架均支持的两种精度训练模式，也是衡量框架性能的混合精度量化训练的重要维度。
   本次测试分别对 FP32 和 AMP 两种精度模式进行了测试。

- **BatchSize**

   本次测试，结合各框架具体情况，BatchSize选用如下：

   | 参数 | PaddlePaddle | NGC TensorFlow 1.15 | NGC PyTorch | NGC 个MXNet |
   |:-----:|:-----:|:-----:|:-----:|:-----:|
   | FP32 | 96 | 128 | 128 | 96 |
   | AMP | 128 | 128 | 128 | 128 |
   | AMP | 208 | 256 | 256 | 192 |

关于其它一些参数的说明：
- **DALI**

   DALI 能够提升数据加载的速度，防止数据加载成为训练的瓶颈。因此，本次测试全部在打开 DALI 模式下进行。

- **XLA**

   本次测试的原则是测试 Resnet50V1.5 在 Paddle 下的最好性能表现，同时对比其与其它框架最好性能表现的优劣。

   因此，对于支持 XLA 的框架，我们默认打开 XLA 模式，以获得该框架最好的吞吐性能数据。

## 二、环境介绍
### 1.物理机环境

- 单机（单卡、8卡）
  - 系统：CentOS Linux release 7.5.1804
  - GPU：Tesla V100-SXM2-16GB * 8
  - CPU：Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz * 38
  - Driver Version: 450.80.02
  - 内存：432 GB

> TODO(Distribute):<br>
> 请李洋提供一下多机环境

- 多机（32卡）
  - 系统：CentOS Linux release 7.5.1804  TODO
  - GPU：Tesla V100-SXM2-32GB * 8
  - CPU：Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz * 40
  - Driver Version: 440.33.01
  - 内存：512 GB

### 2.Docker 镜像

Paddle Docker的基本信息如下：

- Docker: hub.baidubce.com/paddlepaddle/paddle-benchmark:cuda10.1-cudnn7-runtime-ubuntu16.04 TODO
- Paddle：develop+613c46bc0745c8069c55686aef4adc775f9e27d1
- 模型代码：[PaddleClas](https://github.com/PaddlePaddle/PaddleClas)
- CUDA：10.1
- cuDNN：7.6.5

## 三、环境搭建

### 1.单机（单卡、8卡）环境搭建

- 拉取docker
  ```bash
  docker pull hub.baidubce.com/paddlepaddle/paddle-benchmark:cuda10.1-cudnn7-runtime-ubuntu16.04
  ```

- 启动docker
  ```bash
  # 假设imagenet数据放在<path to data>目录下
  nvidia-docker run --shm-size=64g -it -v <path to data>:/data hub.baidubce.com/paddlepaddle/paddle-benchmark:cuda10.1-cudnn7-runtime-ubuntu16.04 /bin/bash
  ```

- 拉取PaddleClas
  ```bash
  git clone https://github.com/PaddlePaddle/PaddleClas.git
  cd PaddleClas
  # 本次测试是在如下版本下完成的：
  git checkout b0904fd250715b3c040c88881395bad06eea9be6
  ```

### 2.多机（32卡）环境搭建

> TODO(Distribute):<br>
> 1. 提供分布式测试环境搭建的详细方法，可参考OneFlow的报告：<br>
> https://github.com/Oneflow-Inc/DLPerf/tree/master/PaddlePaddle/resnet50v1.5#nccl <br>
> https://github.com/Oneflow-Inc/DLPerf/tree/master/NVIDIADeepLearningExamples/TensorFlow/Classification/ConvNets/resnet50v1.5#ssh%E9%85%8D%E7%BD%AE%E5%8F%AF%E9%80%89 <br>
> 2. 注意：咱们Paddle也计划制作Docker镜像，将必要的环境安装在镜像中，如果分布式的环境搭建可以预安装到Docker中，请分布式同学联系王欢，共同制作Docker。而能够在Docker中预安装好的环境，可以在文档的环境搭建介绍中不提供具体安装方法。

- 多机网络部署
```bash
Tesla V100-SXM2-32GB x 8
InfiniBand 100 Gb/sec
Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz
Memory： 512G
Linux version 3.10.0_3-0-0-17
CUDA Version: 10.1, Driver Version: 440.33.01
nvidia-smi topo -m
GPU0    GPU1    GPU2    GPU3    GPU4    GPU5    GPU6    GPU7    mlx5_0  CPU Affinity
GPU0     X      NV2     NV2     NV1     NV1     NODE    NODE    NODE    NODE    0-19
GPU1    NV2      X      NV1     NV1     NODE    NV2     NODE    NODE    NODE    0-19
GPU2    NV2     NV1      X      NV2     NODE    NODE    NV1     NODE    NODE    0-19
GPU3    NV1     NV1     NV2      X      NODE    NODE    NODE    NV2     NODE    0-19
GPU4    NV1     NODE    NODE    NODE     X      NV2     NV2     NV1     NODE    0-19
GPU5    NODE    NV2     NODE    NODE    NV2      X      NV1     NV1     NODE    0-19
GPU6    NODE    NODE    NV1     NODE    NV2     NV1      X      NV2     NODE    0-19
GPU7    NODE    NODE    NODE    NV2     NV1     NV1     NV2      X      NODE    0-19
mlx5_0  NODE    NODE    NODE    NODE    NODE    NODE    NODE    NODE     X

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
  ```

- 数据部署

## 四、测试步骤

### 1.单机（单卡、8卡）测试

- 下载我们编写的测试脚本，并执行该脚本
  ```bash
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/paddle_test_all.sh
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_1gpu_fp32_bs128.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_1gpu_fp32_bs256.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_1gpu_amp_bs128.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_1gpu_amp_bs256.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_8gpu_fp32_bs128.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_8gpu_fp32_bs256.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_8gpu_amp_bs128.yaml
  wget https://raw.githubusercontent.com/wanghuancoder/PerformanceReport/main/ResNet50V1.5/scripts/ResNet50_8gpu_amp_bs256.yaml
  bash paddle_test_all.sh
  ```

- 执行后将得到如下日志文件：
   ```bash
   ./paddle_gpu1_fp32_bs96.txt
   ./paddle_gpu1_amp_bs128.txt
   ./paddle_gpu1_amp_bs208.txt
   ./paddle_gpu8_fp32_bs96.txt
   ./paddle_gpu8_amp_bs128.txt
   ./paddle_gpu8_amp_bs208.txt
   ```

### 2.多机（32卡）测试

> TODO(Distribute):<br>
> 1. 使用PaddleClas中的Resnet50测试32卡分布式性能数据。
> 2. 编写一键执行的测试脚本，可参考： <br>
> https://github.com/Oneflow-Inc/DLPerf#benchmark-test-scopes <br>
> https://github.com/Oneflow-Inc/DLPerf#benchmark-test-scopes <br>

## 五、测试结果

### 1.Paddle训练性能


- 训练吞吐率(images/sec)如下:

|卡数 | FP32(BS=96) | AMP(BS=128) | AMP(BS=208)|
|:-----:|:-----:|:-----:|:-----:|
|1 | 383.0 | 1335.1 | 1400.1 |
|8 | 2753.3 | 8322.9 | 9099.5 |
|32 | - | - | - |

以上数据是根据PaddleClas日志数据，去掉warmup step后，求平均得出。

> TODO(Distribute):<br>
> 完成测试，将32卡数据填入表格

### 2.与业内其它框架对比

说明：
- 同等执行环境下测试
- 单位：`images/sec`
- 对于支持 `DALI/XLA` 的框架，以下测试为开启 `DALI/XLA` 的数据

| 参数 | PaddlePaddle | NGC TensorFlow 1.15 | NGC PyTorch | NGC MXNet |
|:-----:|:-----:|:-----:|:-----:|:-----:|
| FP32 GPU=1,BS=128 | 383(BS=96) | 408.3 | 364.2 | 387.1(BS=96) |
| AMP GPU=1,BS=128 | 1335.1 | 954.9 | 828.7 | 1380.6 |
| AMP GPU=1,BS=256 | 1400.1(BS=208) | 978.8 | 841.6 | 1447.6(BS=192) |
| FP32 GPU=8,BS=128 | 2753.3(BS=96) | 3105.3 | 2826.8 | 2998.1(BS=96) |
| AMP GPU=8,BS=128 | 8322.9 | 7829.9	 | 6014.7 | 9218.9 |
| AMP GPU=8,BS=256 | 9099.5(BS=208) | 7810.8 | 6230.1(BS=248) | 9765.6(BS=192) |
| FP32 GPU=32,BS=128 | -(BS=96) | - | - | -(BS=96) |
| AMP GPU=32,BS=128 | - | - | - | - |
| AMP GPU=32,BS=256 | -(BS=208) | - | - | -(BS=192) |

> 以上测试，由于显存限制，下调了部分测试的BatchSize，并在表格中注明
> Pytorch AMP 8卡在BatchSize=256时会OOM，因此下调BatchSize为248

> TODO(Distribute):<br>
> 完成测试，将32卡数据填入表格

## 六、日志数据
- [1卡 FP32 BS=96 日志](./logs/paddle_gpu1_fp32_bs96.txt)
- [1卡 AMP BS=128 日志](./logs/paddle_gpu1_amp_bs128.txt)
- [1卡 AMP BS=208 日志](./logs/paddle_gpu1_amp_bs208.txt)
- [8卡 FP32 BS=96 日志](./logs/paddle_gpu8_fp32_bs96.txt)
- [8卡 AMP BS=128 日志](./logs/paddle_gpu8_amp_bs128.txt)
- [8卡 AMP BS=208 日志](./logs/paddle_gpu8_amp_bs208.txt)

> TODO(Distribute):<br>
> 完成测试，将32卡 与 公布性能数据 一致的原始日志文件提交到log目录下，并更新链接
