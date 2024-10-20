---
title: GPU服务器开局教程
published: 2024-08-17
description: Fuck Nvidia，Fuck Ubuntu
tags: [技术]
category: 记录
draft: false
---


# GPU服务器开局教程

### 本次实验环境：Ubuntu 22.04 LTS

:::CAUTION
拥有一个良好的网络环境是此类折腾的基础

旁边要有个正常的AI随时可以问
:::
‍

#### 网络代理

1. 最优解决方案设置旁路由，将流量发到旁路由之上
2. 使用xray（[XTLS/Xray-install: Easiest way to install &amp; upgrade Xray. (github.com)](https://github.com/XTLS/Xray-install)，[配置运行 | Project X (xtls.github.io)](https://xtls.github.io/Xray-docs-next/document/config.html#%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE)）


    1. 设置终端走代理（[让终端走代理的几种方法 | 王志文 (zwwangoo.github.io)](https://zwwangoo.github.io/2017/10/17/%E8%AE%A9%E7%BB%88%E7%AB%AF%E8%B5%B0%E4%BB%A3%E7%90%86%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E6%B3%95/)）
    2. 变相实现透明代理

        1. 使用mihomo转发全局流量（[利用Clash进行透明代理的抓包新姿势 - Is Yang&apos;s Blog (isisy.com)](https://www.isisy.com/1530.html)）
        2. 使用tun2socks转发全局流量（[Examples · xjasonlyu/tun2socks Wiki (github.com)](https://github.com/xjasonlyu/tun2socks/wiki/Examples)）
::github{repo="XTLS/Xray-install"}
::github{repo="MetaCubeX/mihomo"}

3. 使用mihomo（[Linux 系统 mihomo 安装教程 - axcsz/Collect GitHub Wiki (github-wiki-see.page)](https://github-wiki-see.page/m/axcsz/Collect/wiki/Linux-%E7%B3%BB%E7%BB%9F-mihomo-%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B)）
4. 谁家用web啊？（[Debian / Ubuntu - v2rayA](https://v2raya.org/en/docs/prologue/installation/debian/)）

#### 驱动安装

* 若为Ubuntu系统，优先使用`sudo ubuntu-drivers`​，在 8卡 V100中`sudo ubuntu-drivers autoinstall`​可自动安装显卡驱动以及cuda环境（好评！！！）

  * 在A100中使用上述安装方法（未成功安装），官方源下载奇慢-->换清华源-->驱动缺失cuda

    * 可能原先环境未清除干净

      * 所使用的卸载命令`sudo apt purge nvidia*`​
      * 但cuda目录，以及环境变量未进行清理（猜测环境变量占主要原因）
      * 也有可能是内核中所加载的驱动未进行卸载

* 使用run方式进行驱动安装

  * 1. Nvidia驱动下载（[下载 NVIDIA 官方驱动 | NVIDIA](https://www.nvidia.cn/drivers/lookup/)）
    2. 删除原有驱动，安装依赖环境（Options）

        1. ​`sudo apt-get remove --purge nvidia*`​
        2. ​`sudo apt install gcc`​
        3. ​`sudo apt install make  #安装驱动需要依赖`​
    3. 禁用nouveau驱动

        1. ```bash
            sudo gedit /etc/modprobe.d/blacklist.conf
            ```
        2. 编辑 /etc/modprobe.d/blacklist-nouveau.conf 文件，末行添加

            ```bash
            blacklist nouveau
            ```
        3. ```bash
            sudo update-initramfs -u #更新
            reboot
            ```
    4. 验证nouveau是否已禁用（没有返回信息显示，说明nouveau已被禁用）

        ```bash
        lsmod | grep nouveau
        ```
    5. 安装nvidia驱动

        ```bash
        sudo chmod a+x NVIDIA-Linux-x86\_64-xxx.xx.run #给文件赋予执行权限
        sudo ./NVIDIA-Linux-x86\_64-xxx.xx.run
        ```
    6. 安装完毕后重启验证

        ```bash
        nvidia-smi
        ```

#### Miniconda3安装

1. 下载安装包[Index of /anaconda/miniconda/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/?C=M&O=D)
2. ```bash
    bash Miniconda3-latest-Linux-x86_64.sh #替换为所下载的sh文件名
    ```
3. 换源（Options）

#### cuda安装

* 若使用第一中安装方式，则不需要进行此步
* 若需要手动安装，则依据[CUDA Toolkit 12.6 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads)进行安装

#### Torch安装

* 以[Start Locally | PyTorch](https://pytorch.org/get-started/locally/)此教程为基础进行安装
* 注意cuda版本与torch的匹配，高版本cuda可以兼容低版本，为避免问题最好与之相等
* 可以直接安装cuda然后torch在虚拟环境里面安装。我不好说跨环境引用情况

#### 性能监控

​`sudo apt install btop`​

​`btop`​

​`pip3 install nvitop`​

​`nvitop -m`​

#### 环境测试

* cuda检测

  ```bash
  nvcc -version
  ```
* Nvidia 系统管理接口

  ```bash
  nvidia-smi
  ```

* Torch检测

  ```python
  import torch
  if torch.cuda.is\_available():
  print("GPU is available")
  else:
  print("GPU is not available")
  ```

#### GPU 测试

* Multi-GPU CUDA stress test（[wilicc/gpu-burn: Multi-GPU CUDA stress test (github.com)](https://github.com/wilicc/gpu-burn)）
  ::github{repo="wilicc/gpu-burn"}

  简单压测意义也就是压测。

  可以参考这个操作 https://www.amaxchina.com/Support/TechDocument/Detail/349

* 一次性测试所有环境情况

  ​`conda create test`​

  ​`conda activate test`​

  ​`mkdir test`​

  ​`cd test`​

  ​`nano testspeed.py`​

  ```python
  import torch
  import time
  # 这个是娱乐局测速，没什么实际意义，但是可以测试多卡互联情况。
  # 获取所有可用的GPU设备
  device_count = torch.cuda.device_count()

  # 目标显存占用量
  target_memory_usage_gb = 30
  target_memory_usage_bytes = target_memory_usage_gb * 1024**3

  # 计算每个张量的大小（假设float32，每个元素4字节）
  dtype_size_in_bytes = 4  # float32
  elements_per_tensor = target_memory_usage_bytes // dtype_size_in_bytes // device_count  # 每个GPU分配部分

  # 确定张量的形状（假设为2D张量）
  tensor_shape = (int(elements_per_tensor ** 0.5), int(elements_per_tensor ** 0.5))

  # 在每个GPU上创建张量，达到目标显存占用
  tensors = [torch.randn(tensor_shape, dtype=torch.float32, device=f'cuda:{i}') for i in range(device_count)]

  # 开始计时
  start_time = time.time()
  duration = 60  # 秒

  try:
      while time.time() - start_time < duration:
          for i in range(device_count):
              # 记录开始时间
              calc_start_time = time.time()
            
              # 对每个GPU上的张量进行矩阵乘法，保持高计算负载
              result = torch.matmul(tensors[i], tensors[i])
            
              # 同步操作，确保计算完成
              torch.cuda.synchronize()
            
              # 计算时间和速度
              calc_duration = time.time() - calc_start_time
              num_flops = 2 * tensor_shape[0] * tensor_shape[1] * tensor_shape[1]  # 矩阵乘法 FLOPs 计算
              flops_per_sec = num_flops / calc_duration / 1e12  # 转换为 TFLOPs

              print(f"GPU {i} 计算速度: {flops_per_sec:.2f} TFLOPs, 耗时: {calc_duration:.6f} 秒")

  except RuntimeError as e:
      print(f"计算时遇到错误: {e}")

  print(f"计算测试完成，已运行 {duration} 秒。")
  ```

  ​`python3 testspeed.py`​

  ​`nano testpower.py`​

  ```python
  import torch
  import time
  import pynvml
  # 这个主要是为了持续性压力测试，至于显存我没想占满。
  # 初始化pynvml以监控GPU功率
  pynvml.nvmlInit()

  # 获取所有可用的GPU设备
  device_count = torch.cuda.device_count()

  # 目标显存占用量（记得修改这个）！
  target_memory_usage_gb = 30
  target_memory_usage_bytes = target_memory_usage_gb * 1024**3

  # 数据类型的字节数
  dtype_size_in_bytes = 4  # float32

  # 计算每个GPU上的张量数目
  num_tensors = 3  # 假设创建3个张量来达到目标显存占用
  elements_per_tensor = target_memory_usage_bytes // dtype_size_in_bytes // device_count // num_tensors

  # 确定张量的形状（假设为2D张量）
  tensor_shape = (int(elements_per_tensor ** 0.5), int(elements_per_tensor ** 0.5))

  # 在每个GPU上创建多个张量，达到目标显存占用
  tensors = []
  for i in range(device_count):
      gpu_tensors = [torch.randn(tensor_shape, dtype=torch.float32, device=f'cuda:{i}') for _ in range(num_tensors)]
      tensors.append(gpu_tensors)

  # 检查显存占用
  for i in range(device_count):
      torch.cuda.synchronize()
      print(f"GPU {i} 当前显存占用: {torch.cuda.memory_allocated(i) / (1024**3):.2f} GB")

  # 开始计时
  start_time = time.time()
  duration = 1600  # 秒，也就是测试时间

  try:
      while time.time() - start_time < duration:
          elapsed_time = time.time() - start_time
          remaining_time = duration - elapsed_time
        
          print(f"剩余时间: {remaining_time:.1f} 秒")
        
          for i in range(device_count):
              # 获取GPU功率
              handle = pynvml.nvmlDeviceGetHandleByIndex(i)
              power_usage = pynvml.nvmlDeviceGetPowerUsage(handle) / 1000  # 转换为瓦特
            
              print(f"GPU {i} 功率: {power_usage:.2f} W")
            
              for tensor in tensors[i]:
                  # 对每个GPU上的张量进行矩阵乘法，保持高计算负载
                  result = torch.matmul(tensor, tensor)
              # 确保计算完成
              torch.cuda.synchronize()
        
          time.sleep(0.001)  # 每秒输出一次倒计时和功率

  except RuntimeError as e:
      print(f"计算时遇到错误: {e}")

  print(f"计算测试完成，已运行 {duration} 秒。")

  # 释放pynvml资源
  pynvml.nvmlShutdown()

  ```

  ​`python3 testpower.py`​

#### 新遇到的问题

* cuda runtime error (802) : system not yet initialized .../THCGeneral.cpp:50
  
  > [!TIP]
  > （由题目自拟闯天涯实操过程中发现的问题）
  
  ::github{repo="pytorch/pytorch"}
  该issue提供了解决方法 （[CUDA 运行时错误 （802）：系统尚未初始化 .../THCGeneral.cpp：50 ·问题 #35710 ·pytorch/pytorch --- cuda runtime error (802) : system not yet initialized .../THCGeneral.cpp:50 · Issue #35710 · pytorch/pytorch](https://github.com/pytorch/pytorch/issues/35710)）

  * 尝试编译并运行 [NVIDIA/cuda-samples: Samples for CUDA Developers which demonstrates features in CUDA Toolkit](https://github.com/NVIDIA/cuda-samples)
  ::github{repo="NVIDIA/cuda-samples"}

  * ```zsh
    git clone https://github.com/NVIDIA/cuda-samples.git
    cd cuda-samples/Samples/bandwidthTest
    make
    ./bandwidthTest
    ```
  * > [!NOTE] 
    > `nvcc`​ is going to be located in `/usr/local/cuda/bin`​ 注意： nvcc 将位于 /usr/local/cuda/bin
  * 运行后结果：

    * 未安装 Data Center GPU 管理器

      * ```zsh
        > ./bandwidthTest
        [CUDA Bandwidth Test] - Starting...
        Running on...

        cudaGetDeviceProperties returned 802
        -> system not yet initialized
        CUDA error at bandwidthTest.cu:256 code=802(cudaErrorSystemNotReady) "cudaSetDevice(currentDevice)" 
        ```
      * 意味着未安装 Data Center GPU 管理器
      * ```zsh
        wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
        sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
        sudo apt-key adv --keyserver-options http-proxy=http://proxy-chain.intel.com:911  --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
        ```

        * 若出现 unable to fetch （~~网络环境好应该不会出现~~

          * ```zsh
            >  sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
            Executing: /tmp/apt-key-gpghome.qjhmgicscb/gpg.1.sh --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
            gpg: requesting key from 'https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub'
            gpg: WARNING: unable to fetch URI https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub: Connection timed out
            ```
          * 那就设置代理

            ```zsh
            sudo apt-key adv --keyserver-options http-proxy=<PROXY-ADDRESS:PORT>  --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
            ```
      * 然后开始安装 datacenter-gpu-manager

        ```zsh
        sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86\_64/ /"
        sudo apt-get update
        sudo apt-get install -y datacenter-gpu-manager
        ```
      * 终止主机引擎

        ```zsh
        sudo nv-hostengine -t
        ```
      * 启动 fabricManager

        ```zsh
        sudo service nvidia-fabricmanager start
        ```

        * 如果出现

          ```zsh
          > sudo service nvidia-fabricmanager start
          Failed to start nvidia-fabricmanager.service: Unit nvidia-fabricmanager.service not found.
          ```
        * 安装 Fabric Manager并启动

          ```zsh
          sudo apt-get install cuda-drivers-fabricmanager-<version>
          sudo service nvidia-fabricmanager start
          ```
  * 再次运行`bandwidthTest`​

    ```zsh
    > ./bandwidthTest
    [CUDA Bandwidth Test] - Starting...
    Running on...

     Device 0: NVIDIA A100-SXM4-40GB
     Quick Mode

     Host to Device Bandwidth, 1 Device(s)
     PINNED Memory Transfers
       Transfer Size (Bytes)        Bandwidth(GB/s)
       32000000                     26.1

     Device to Host Bandwidth, 1 Device(s)
     PINNED Memory Transfers
       Transfer Size (Bytes)        Bandwidth(GB/s)
       32000000                     25.6

     Device to Device Bandwidth, 1 Device(s)
     PINNED Memory Transfers
       Transfer Size (Bytes)        Bandwidth(GB/s)
       32000000                     1152.7

    Result = PASS

    NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.

    > ipython #torch检测
    Python 3.7.11 (default, Jul 27 2021, 14:32:16) 
    Type 'copyright', 'credits' or 'license' for more information
    IPython 7.26.0 -- An enhanced Interactive Python. Type '?' for help.

    In [1]: import torch
    tor
    In [2]: torch.cuda.is_available()
    Out[2]: True
    ```

#### 具体应用

* ollama（已实现，详见另一篇文章）
* vllm（待实现，据说对多卡推理优化极好）

‍
