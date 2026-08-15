---
title: 下个CUDA版真费劲
description: 下了清华的，发现没用CUDA只有CPU版本，转阿里云发现还有参数限制，不然下载的也是错的，记录一下
pubDate: 2026-08-15
category: learning
tags: ['技术']
draft: false
---


# 关于我的PyTorch GPU 版安装全流程记录

> 目标：在 Windows 11 上使用 Python 3.13（Anaconda 环境），目标安装带 CUDA 12.8 支持的 PyTorch GPU 版本。

---
## 说点什么

本来也没准备用anaconda包，但我的 miniconda 装 wsl 上了拉不下来。然后我就直接用 windows 侧的了，没准备再连一下 wsl。结果弄了半天没弄好。终于结束，叫AI做总结了一下，我真要考虑以后都用 agent 自动装环境了。但我现在还没有找到喜欢的 computer-use，hermes 上我装到 Ubuntu 了，导致连不上 Windows。有人看到能给我推荐一个不那么重的吗。。。

## 一、环境信息

| 项目 | 值 |
|:---|:---|
| 操作系统 | Windows 11 |
| Python 版本 | 3.13 |
| 包管理器 | pip（Anaconda 自带） |
| 目标 CUDA 版本 | 12.8 (cu128) |
| Anaconda 路径 | `C:\Users\LENOVO\anaconda3` |

---

## 二、安装步骤

### 步骤 1：查找可用版本

通过访问阿里云 PyTorch 镜像站 `https://mirrors.aliyun.com/pytorch-wheels/cu128/`，查到 Python 3.13 + Windows + CUDA 12.8 下最新可用版本：

| 包名 | 版本号 |
|:---|:---|
| torch | 2.11.0+cu128 |
| torchvision | 0.26.0+cu128 |
| torchaudio | 2.11.0+cu128 |

### 步骤 2：执行安装命令

```powershell
C:\Users\LENOVO\anaconda3\python.exe -m pip install torch==2.11.0+cu128 torchvision==0.26.0+cu128 torchaudio==2.11.0+cu128 -f https://mirrors.aliyun.com/pytorch-wheels/cu128/
```

### 步骤 3：验证安装

```powershell
C:\Users\LENOVO\anaconda3\python.exe -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

预期输出：`2.11.0+cu128` 和 `True`。

---

## 三、踩到的所有坑

### 坑 1：镜像源参数用错，装出 CPU 版

**现象**：使用清华镜像源，安装后发现是 CPU 版本，没有 CUDA 支持。

**原因**：清华镜像有两个不同的地址：

- 通用 PyPI 镜像（`https://pypi.tuna.tsinghua.edu.cn/simple`）：用 `-i`（`--index-url`）指定时，默认匹配到的是 CPU 版本的 PyTorch。
- PyTorch 专用镜像（`https://mirrors.tuna.tsinghua.edu.cn/pytorch-wheels/cu128`）：这才是包含 GPU/CUDA 版本的正确地址。

用 `-i` 参数会**替换**默认索引源，可能匹配到 CPU 包；而用 `-f`（`--find-links`）参数是**追加**查找链接，从指定目录找 GPU 版，不会覆盖默认索引。

**教训**：安装 GPU 版 PyTorch 时，务必使用 `-f`（`--find-links`）参数指向 PyTorch 专用镜像目录，不要用 `-i` 指向通用 PyPI 镜像。

---

### 坑 2：不指定版本号，pip 自动匹配到 CPU 版

**现象**：执行 `pip install torch torchvision torchaudio -f https://mirrors.aliyun.com/pytorch-wheels/cu128/`（不指定版本号），pip 自动匹配到了 `2.13.0` 的 CPU 版本。

**原因**：不指定版本号时，pip 会优先匹配最高版本，但该版本在镜像中可能只有 CPU 构建，于是自动安装了 CPU 版。

**教训**：必须显式指定 `+cu128` 后缀的版本号，如 `torch==2.11.0+cu128`，否则 pip 可能再次匹配到 CPU 版本。

---

### 坑 3：系统中有多个 Python 环境，`python` 命令指向了错误的环境

**现象**：安装成功后，终端执行 `python -c "import torch"` 报错 `ModuleNotFoundError: No module named 'torch'`。

**诊断**：运行 `where.exe python` 发现系统中有三个 Python：

| 优先级 | 路径 | 说明 |
|:---|:---|:---|
| 1 | `C:\Users\LENOVO\AppData\Local\Programs\Python\Python313\python.exe` | 独立安装的 Python 3.13 |
| 2 | `...\hermes-agent\venv\Scripts\python.exe` | hermes 项目的虚拟环境 |
| 3 | `...\Microsoft\WindowsApps\python.exe` | Windows 应用商店的 Python |

Anaconda 的路径 `C:\Users\LENOVO\anaconda3\python.exe` **完全不在 PATH 中**。

**原因**：Anaconda 未加入系统 PATH，终端直接输入 `python` 时，系统按 PATH 顺序优先找到了其他解释器（独立安装的 Python 3.13），而这个环境里并没有安装 torch。

**教训**：安装和运行必须使用同一个 Python 环境。最稳妥的做法是始终使用完整路径调用 Anaconda 的 Python 解释器。

---

### 坑 4：先装了 CPU 版再装 GPU 版，pip 提示"已满足要求"不重新安装

**现象**：先安装了 CPU 版（2.13.0），然后执行 GPU 版安装命令，pip 输出 `Requirement already satisfied`，没有重新安装 GPU 版。

**原因**：pip 发现已经安装了满足要求的版本（CPU 版 2.13.0），但 `-f` 参数只是追加查找链接，并不会强制替换已安装的包。

**教训**：必须先卸载已有的 CPU 版本，再安装 GPU 版本：

```powershell
C:\Users\LENOVO\anaconda3\python.exe -m pip uninstall torch torchvision torchaudio -y
```

---

### 坑 5：安装和验证用了不同的 Python，反复报 `ModuleNotFoundError`

**现象**：用 `C:\Users\LENOVO\anaconda3\python.exe` 安装成功后，用 `python` 验证时又报 `ModuleNotFoundError`。

**原因**：`python` 指向的是 `Python313`，而 torch 安装在 Anaconda 环境中。两个环境完全独立。

**教训**：安装命令和验证命令必须使用**完全相同**的 Python 路径。

---

## 四、最终正确的操作流程

### 1. 卸载旧的 CPU 版本

```powershell
C:\Users\LENOVO\anaconda3\python.exe -m pip uninstall torch torchvision torchaudio -y
```

### 2. 安装 GPU 版本

```powershell
C:\Users\LENOVO\anaconda3\python.exe -m pip install torch==2.11.0+cu128 torchvision==0.26.0+cu128 torchaudio==2.11.0+cu128 -f https://mirrors.aliyun.com/pytorch-wheels/cu128/
```

### 3. 验证安装

```powershell
C:\Users\LENOVO\anaconda3\python.exe -c "import torch; print('版本:', torch.__version__); print('CUDA:', torch.version.cuda); print('GPU可用:', torch.cuda.is_available())"

```

预期输出：

```
版本: 2.11.0+cu128
CUDA: 12.8
GPU可用: True
```
