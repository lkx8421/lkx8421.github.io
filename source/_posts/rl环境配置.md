---
title: rl环境配置
date: 2025-02-05 18:10:17
tags: 
    - reinforcement learning
    - CUDA
categories: rl
description: 配置rl仿真环境，用来四足或者双足机器人训练
---

# 硬件要求以及驱动安装

## 硬件要求

显卡准备，3060以上，12GB显存以上

## 驱动安装

可以参考以下文档

https://zhuanlan.zhihu.com/p/560826876

![](<2024-06-07 14-08-02 的屏幕截图-1.png>)

![](<2024-06-07 14-08-02 的屏幕截图.png>)

### nvidia驱动安装

https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin\_46584887/article/details/122726265

安装完成后用nvidia-smi命令(查看nvidia驱动)，“CUDA Version”提示安装cuda版本的上限，它是12.2，

所以我选择安装12.0

### cuda安装

https://link.zhihu.com/?target=https%3A//developer.nvidia.com/cuda-toolkit-archive

Cuda installation guide:https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu

![](image.png)

安装完成后用nvcc -V(查看cuda版本)命令查看

![](<2024-06-07 11-56-20 的屏幕截图.png>)

### anaconda的安装

https://zhuanlan.zhihu.com/p/32925500，

https://docs.anaconda.com/anaconda/install/

![](<2024-09-07 11-06-41 的屏幕截图.png>)

conda相关的命令稍后会介绍，它能够建立一个虚拟环境，配置相应的python版本以及cuda-toolkit版本，在不影响本机版本的条件下保证算法和环境的适配。

# 安装issacgym

https://developer.nvidia.com/isaac-gym/download

![](<2024-09-07 10-28-08 的屏幕截图.png>)

安装后解压即可

# conda配置虚拟环境

```plain&#x20;text
conda create -n your_env_name python=3.8
```

该环境配置，你能在你的路径/anaconda3/envs/your\_env\_name找到

之后，激活环境能在终端开头看到“（your\_env\_name）”

```plain&#x20;text
conda activate your_env_name
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:your_path/anaconda3/envs/your_env_name/lib
```

# 一个简单的测试

```plain&#x20;text
pip3 install torch==1.10.0+cu113 torchvision==0.11.1+cu113 torchaudio==0.10.0+cu113 -f https://download.pytorch.org/whl/cu113/torch_stable.html
```

```plain&#x20;text
cd 你的路径/isaacgym/python && pip install -e .
```

```plain&#x20;text
cd examples && python 1080_balls_of_solitude.py
```

看到一堆球落到地上表示成功

**注意：如果不跑强化学习算法和强化学习环境，记得关闭虚拟环境，不然影响ros或者gazebo的正常运行**

conda deactivate

## possible problems

https://zhuanlan.zhihu.com/p/685981407

### “Isaac Gym”没有反应

运行示例代码（例如 joint\_monkey.py）时，Isaac Gym 屏幕可能不会显示。 这是因为一些集成英特尔显卡（例如英特尔 UHD 显卡）的 PC 系统中并未使用 NVIDIA 的 GPU。 请输入以下命令以使用 NVIDIA GPU。

```shell
sudo prime-select nvidia
export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json
```
