---
title: "Ulzix半周年庆免费NAT VPS测评"
date: 2026-08-02T18:57:54+08:00
categories: 计算机
---
## 前言

最近入手了一台Ulzix的半周年庆的免费香港NAT VPS，这台 VPS 的配置并不高，因此本文主要记录它的实际性能表现，以及它适合什么用途。

测试环境：

- 系统： Alpinelinux 3.22 x86_64 (20251224_0050)
- 虚拟化：LXC
- 地区：中国香港
- 测试工具：YABS

## 基本配置

```bash
curl -sL yabs.sh | bash
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2026-07-24                    #
# https://github.com/masonr/yet-another-bench-script #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

Sun Aug  2 10:52:48 UTC 2026

Warning: locale 'C' not detected. Test outputs may not be parsed correctly.

Basic System Information:
---------------------------------
Uptime     : 0 days, 0 hours, 8 minutes
Processor  : Intel(R) Xeon(R) CPU E5-2696 v4 @ 2.20GHz
CPU cores  : 1 @ 2199.998 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
RAM        : 122.1 MiB
Swap       : 122.1 MiB
Disk       :
Distro     : Alpine Linux v3.22
Kernel     : 6.1.0-50-amd64
VM Type    :
IPv4/IPv6  : ✔ Online / ❌ Offline
```
得到基础信息。CPU 为 Intel Xeon E5-2696 v4。这是一款较老的服务器处理器，发布于 Broadwell 时代。虽然年代较久，但单核 2.2GHz 对于轻量服务来说仍然够用。这台VPS的局限在内存。不到 256MB 的内存对于现代应用来说非常有限。因此使用时需要尽量选择轻量方案。Alpine Linux 在这里发挥了优势，占用资源较少。

## 网络测试

YABS 网络测试结果：

```bash
IPv4 Network Information:
---------------------------------
ISP        : cognetcloud INC
ASN        : AS401696 cognetcloud INC
Host       : GA CLOUD TECHNOLOGY (HONG KONG) LIMITED
Location   : Ho Man Tin, Kowloon City (KKC)
Country    : Hong Kong

Less than 2GB of space available. Skipping disk test...

iperf3 Network Speed Tests (IPv4):
---------------------------------
Provider        | Location (Link)           | Send Speed      | Recv Speed      | Ping
-----           | -----                     | ----            | ----            | ----
Clouvider       | London, UK (10G)          | busy            | busy            | 189.875 ms
Eranium         | Amsterdam, NL (100G)      | busy            | busy            | 242.073 ms
Uztelecom       | Tashkent, UZ (10G)        | busy            | busy            | 228.469 ms
Leaseweb        | Singapore, SG (10G)       | busy            | busy            | 41.411 ms
Clouvider       | Los Angeles, CA, US (10G) | busy            | busy            | 144.616 ms
Leaseweb        | NYC, NY, US (10G)         | busy            | busy            | 219.914 ms
Edgoo           | Sao Paulo, BR (1G)        | busy            | busy            | 322.761 ms
```

节点位于香港，因此亚洲方向延迟较低,欧美方向延迟较高。对于个人博客、代理反代、小工具来说，亚洲线路表现可以接受。

## IP质量：
![ulzix-free-nat-vps-ip-1.png](https://pub-aa95b769ea0843048c4d3181378b7b3c.r2.dev/ulzix-free-nat-vps-ip-1.png)
![ulzix-free-nat-vps-ip-2.png](https://pub-aa95b769ea0843048c4d3181378b7b3c.r2.dev/ulzix-free-nat-vps-ip-2.png)
![ulzix-free-nat-vps-ip-3.png](https://pub-aa95b769ea0843048c4d3181378b7b3c.r2.dev/ulzix-free-nat-vps-ip-3.png)
## 磁盘测试

YABS 没有进行磁盘测试：
```bash
Less than 2GB of space available.
Skipping disk test...
```
由于剩余空间不足 2GB，脚本自动跳过了 fio 测试。

## Geekbench 测试
Geekbench 测试失败：
```bash
Geekbench test failed and low memory was detected.
```
原因也很明显：这台 VPS 只有 122MB 内存，无法满足 Geekbench 的运行需求。

## 总结：

经过测试，这台 VPS 更适合作为个人博客，轻量 Web 服务，学习 Linux。由于内存限制，不推荐：Docker 大量部署，数据库服务，Node.js 大型项目，编译大型项目，游戏服务器。

这台香港 NAT VPS 并不是性能型机器。它的优势：免费！香港节点，亚洲延迟不错，Alpine 系统轻量，足够运行简单服务。

缺点：内存只有约 122MB，磁盘空间较紧张，性能有限。

如果把它看作一台“小型 Linux 实验机”，它还是有价值的。

对于个人博客、轻量服务和 Linux 学习来说，这个NAT VPS仍然是一个不错的选择。感谢Ulzix提供免费NAT VPS！