---
title: GROMACS AMD GPU 编译指南（ROCm）
date: 2026-07-15
updated: 2026-07-15
tags: [GROMACS, AMD, ROCm, HIP, GPU, 编译]
categories: HPC
keywords: GROMACS,AMD,ROCm,GPU,编译,HIP
description: 使用 AMD ROCm 编译 GROMACS 的 cmake 配置与参数说明
cover: https://picsum.photos/id/60/800/450
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
---

## 1. 环境

| 组件 | 版本/路径 |
|------|----------|
| ROCm | 7.2.4，安装于 `/opt/rocm-7.2.4` |
| 编译器 | `amdclang` / `amdclang++` |
| GPU 架构 | gfx1031（RDNA2，对应 RX 6700 XT 等） |
| GROMACS | 需通过 `-DGMX_GPU=HIP` 启用 GPU 加速 |
| FFT 库 | VkFFT（GPU FFT）+ 自带 FFTW（`-DGMX_BUILD_OWN_FFTW=ON`） |

## 2. cmake 参数总览

```bash
cmake .. \
    -DCMAKE_C_COMPILER=/opt/rocm-7.2.4/bin/amdclang \
    -DCMAKE_CXX_COMPILER=/opt/rocm-7.2.4/bin/amdclang++ \
    -DCMAKE_HIP_COMPILER=/opt/rocm-7.2.4/bin/amdclang++ \
    -DCMAKE_PREFIX_PATH=/opt/rocm-7.2.4 \
    -DGMX_GPU=HIP \
    -DGMX_HIP_TARGET_ARCH=gfx1031 \
    -DGMX_GPU_FFT_LIBRARY=VkFFT \
    -DGMX_BUILD_OWN_FFTW=ON \
    -DREGRESSIONTEST_DOWNLOAD=ON \
    -DCMAKE_INSTALL_PREFIX=~/software/gromacs
```

## 3. 参数说明

| 参数 | 说明 |
|------|------|
| `CMAKE_C_COMPILER` | 指定 C 编译器为 ROCm 自带的 `amdclang` |
| `CMAKE_CXX_COMPILER` | 指定 C++ 编译器为 `amdclang++` |
| `CMAKE_HIP_COMPILER` | 指定 HIP 编译器，与 C++ 编译器一致 |
| `CMAKE_PREFIX_PATH` | ROCm 安装根目录，cmake 会在此路径下查找 `hip`、`rocfft` 等库 |
| `GMX_GPU` | GPU 后端类型，AMD 平台设为 `HIP` |
| `GMX_HIP_TARGET_ARCH` | GPU 目标架构代号，需根据实际显卡填写（见下方对照表） |
| `GMX_GPU_FFT_LIBRARY` | GPU FFT 库，可选 `VkFFT` 或 `rocFFT`；VkFFT 性能通常更优 |
| `GMX_BUILD_OWN_FFTW` | 使用 GROMACS 自带的 FFTW，无需单独安装 |
| `REGRESSIONTEST_DOWNLOAD` | 自动下载回归测试数据，方便编译后验证 |
| `CMAKE_INSTALL_PREFIX` | 安装路径 |

## 4. GPU 架构代号对照

| 架构代号 | GPU 系列 | 典型型号 |
|----------|---------|---------|
| `gfx906` | Vega 20 | Radeon VII |
| `gfx908` | CDNA1 | MI50 / MI100 |
| `gfx90a` | CDNA2 | MI210 / MI250X |
| `gfx942` | CDNA3 | MI300X |
| `gfx1030` | RDNA2 | RX 6900 XT / RX 6800 XT |
| `gfx1031` | RDNA2 | RX 6700 XT |
| `gfx1100` | RDNA3 | RX 7900 XTX |

查看当前 GPU 架构代号：

```bash
rocminfo | grep -E "Name:|gfx"
```

## 5. 编译步骤

```bash
mkdir build && cd build

cmake .. \
    -DCMAKE_C_COMPILER=/opt/rocm-7.2.4/bin/amdclang \
    -DCMAKE_CXX_COMPILER=/opt/rocm-7.2.4/bin/amdclang++ \
    -DCMAKE_HIP_COMPILER=/opt/rocm-7.2.4/bin/amdclang++ \
    -DCMAKE_PREFIX_PATH=/opt/rocm-7.2.4 \
    -DGMX_GPU=HIP \
    -DGMX_HIP_TARGET_ARCH=gfx1031 \
    -DGMX_GPU_FFT_LIBRARY=VkFFT \
    -DGMX_BUILD_OWN_FFTW=ON \
    -DREGRESSIONTEST_DOWNLOAD=ON \
    -DCMAKE_INSTALL_PREFIX=~/software/gromacs

make -j$(nproc)
make install
```

提示：编译完成后建议运行回归测试验证 GPU 加速是否正常工作：

```bash
make check
```
