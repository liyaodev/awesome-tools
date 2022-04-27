# 3. 深度总结，带你玩转 NVIDIA GPU

大家好，我是`三十一`\[0]，最近北京也出现疫情了，昨晚公司大楼临时管控，测核酸折腾到小一点才到家。前两天的抢菜、囤菜，加上这次的管控经历，这次真有些慌了。。。

本次分享的内容比较简单，主要是对日常工作使用 GPU 的常用命令做一个简单的总结，阅读全文预计花费 11 分钟，如果有缺失号友们可以私信我补充（划重点），如果对你有帮助，也欢迎号友们点赞收藏。

### nvidia-smi 简介

NVIDIA 系统管理界面（nvidia-smi）是基于 `NVIDIA Management Library（NVML）`\[1]的命令行实用程序，旨在帮助管理和监视 NVIDIA GPU 设备。

### GPU 参数查看

#### 一、查看 GPU 运行情况

```shell
nvidia-smi
```

```shell
Sun Mar 28 02:40:38 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.56       Driver Version: 418.56       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  On   | 00000000:02:00.0 Off |                  N/A |
| 23%   29C    P8     9W / 250W |    611MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 108...  On   | 00000000:03:00.0 Off |                  N/A |
| 23%   30C    P8     9W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  GeForce GTX 108...  On   | 00000000:82:00.0 Off |                  N/A |
| 23%   30C    P8     9W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  GeForce GTX 108...  On   | 00000000:83:00.0 Off |                  N/A |
| 23%   30C    P8     9W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     33777      C   /usr/bin/python                              601MiB |
+-----------------------------------------------------------------------------+
```

以上是`GEFORCE GTX 1080 Ti` GPU 服务器的运行信息。

* 第一行分别为：命令行工具版本、GPU驱动版本、CUDA版本
* 第一栏分别为：GPU(GPU卡号，0～4)、Fan(风扇转速，0～100%)
* 第二栏分别为：Name(显卡名字)、Temp(温度，摄氏度)
* 第三栏分别为：Perf(性能状态，P0\~P12，最高性能为P0，最低性能为P12)
* 第四栏分别为：Persistence-M(持续模式，默认为关闭，比较节能，如果设置成ON，耗能比较大，但新的GPU应用启动时，花费的时间会更短)、Pwr:Usage/Cap(能耗)
* 第五栏分别为：Bus-Id(GPU总线，domain:bus:device.function)
* 第六栏分别为：Disp.A(GPU的显示是否初始化)、Memory-Usage(显存利用率)
* 第七栏分别为：Volatile GPU-Util(GPU浮动利用率)
* 第八栏分别为：Uncorr. ECC(Error Correcting Code错误检查和纠正码)、Compute M.(计算模式)
* 下面一张表为：每个GPU Processes的资源占用情况

**注**：显存占用和 GPU 占用是两个不一样的，显卡是由 GPU 和显存等组成的，显存和 GPU 的关系可简单理解为内存和 CPU 的关系。

当然我们也可以每秒刷新查询一次，实现实时监控查询显卡状态效果

```shell
watch -n 1 nvidia-smi

或

nvidia-smi -l 1
```

#### 二、查询所有 GPU 的当前详细信息

```shell
nvidia-smi -q
```

也可以单独过滤第 N 卡 的 GPU 信息

```shell
nvidia-smi -q -i 0
```

或者单独过滤当前的 GPU 时钟相关信息

```shell
nvidia-smi -q -d CLOCK
```

或者单独过滤每个 GPU 的可用时钟频率信息

```shell
nvidia-smi -q -d SUPPORTED_CLOCKS
```

#### 三、获取 GPU ID 信息

```shell
nvidia-smi -L
```

从左到右分别为：GPU卡号、GPU型号、GPU物理UUID号

```shell
GPU 0: GeForce GTX 1080 Ti (UUID: GPU-5da6e67e-fd5a-88fb-7a0e-109c3284f7bf)
GPU 1: GeForce GTX 1080 Ti (UUID: GPU-ce9189e4-2e58-3a19-4332-cb5c7fac1aa6)
GPU 2: GeForce GTX 1080 Ti (UUID: GPU-242b3020-8e5c-813a-42d9-475766d52f9d)
GPU 3: GeForce GTX 1080 Ti (UUID: GPU-8f3d825f-7246-3daf-eaa1-37845b03aa03)
```

可以单独过滤出 GPU 卡号信息

```shell
nvidia-smi -L | cut -d ' ' -f 2 | cut -c 1
```

### NVIDIA Docker 运行相关

```shell
docker run --runtime nvidia -e NVIDIA_VISIBLE_DEVICES=GPU-5da6e67e-fd5a-88fb-7a0e-109c3284f7bf -p 8000:8000 -v /www/models:/www/models -it --rm <项目/服务:v1.0.0>
```

* \--runtime nvidia：指定 NVIDIA GPU 驱动运行运行，只针对 GPU 机器生效。
* \-e NVIDIA\_VISIBLE\_DEVICES：指定物理机分配给 Docker 程序的 GPU 卡设备，常用可选值如下：
  * GPU-fef8089b 或 多个 UUID 逗号分隔：指定挂载具体 GPU 卡设备
  * all：指定挂载当前机器所有卡

### GPUtil Python 组件

GPUtil 是一个 Python 模块，支持 Python 2.X 和 3.X。用于在 Python 中以编程方式使用 nvidia-smi 从 NVIDA GPU 获取 GPU 状态。

可以通过 pip 命令进行直接`GPUtil 安装`\[2]

```shell
pip install GPUtil
```

查询 GPU 当前的使用情况

```shell
import GPUtil
GPUtil.showUtilization()
```

根据当前内存使用情况和负载确定的，自动选择合适的 GPU 卡

```shell
deviceIDs = GPUtil.getAvailable(order = 'first', limit = 1, maxLoad = 0.5, maxMemory = 0.5, includeNan=False, excludeID=[], excludeUUID=[])
```

核心参数解释：

* order：确定返回可用 GPU 设备 ID 的排序，具体如下：
  * first：按升序排列可用的 GPU 设备 ID（默认）
  * last：按降序排列可用的 GPU 设备 ID
  * random：随机订购可用的 GPU 设备 ID
  * load：按负载递增排序可用的 GPU 设备 ID
  * memory：通过升序内存使用来排序可用的 GPU 设备 ID
* limit：将返回的 GPU 设备 ID 数量限制为指定数量，必须是正整数。（默认 = 1）
* maxLoad：被认为可用的 GPU 的最大当前相对负载。负载大于 的 GPUmaxLoad不会返回。（默认 = 0.5）
* maxMemory：被视为可用的 GPU 的最大当前相对内存使用量。maxMemory不返回当前内存使用量大于的 GPU 。（默认 = 0.5）
* excludeID：ID 列表，应从可用 GPU 列表中排除。见GPU类描述。（默认 = \[]）

### GPU 常用设置

#### 一、启动模式设置

针对 NVIDIA 首次运行启动加载慢，我们可以进行 `Persistence-M` 持续模式设置，如下：

```shell
sudo nvidia-smi -pm 1
```

#### 二、节点分配设置

针对四卡机器或多卡机器，偶尔会出现卡性能不均匀现象。一个纯野生小技巧，我们可以优先选用边界节点，边界卡槽有利于散热。

### References

* \[0] 三十一: http://www.lee31.cn/assets/image/ThirtyOneLee.jpeg
* \[1] NVIDIA Management Library（NVML）: https://developer.nvidia.com/nvidia-system-management-interface
* \[2] GPUtil 安装: https://pypi.org/project/GPUtil/

### 往期文章

* 还活在上个时代，Etcd 3.0 实现分布式锁竟如此简单
* 从 0 到 1，如何徒手撸一个 Python 插件系统？
* 一站式机器学习开业平台 MLflow 怎么样？
* 开源项目 requests 的 stars 为啥比 python 还多 3.7k？
* 业余不求人，30秒AI快速制作LOGO
* 学习Protobuf，ZigZag是啥你真的知道么？
