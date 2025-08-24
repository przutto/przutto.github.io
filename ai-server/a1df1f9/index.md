# BIOS优化


这是本系列的BIOS优化篇，参考了 AMD 官方文档，旨在帮助您优化基于 **AMD EPYC™ 7002 系列处理器**的系统 BIOS 设置，以获得最佳的 AI 工作负载性能。

<!--more-->
---
参考文档：

[High Performance Computing (HPC) Tuning Guide for AMD EPYC™ 7002 Series Processors](https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/tuning-guides/amd-epyc-7002-tg-hpc-56827.pdf)

[AMD EPYC™ 9005 BIOS & WORKLOAD TUNING GUIDE](https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/tuning-guides/58467_amd-epyc-9005-tg-bios-and-workload.pdf)


---

### AI 工作负载的特点与优化目标

AI 工作负载，特别是深度学习，对计算资源有极高的要求，主要体现在：

* **高浮点计算：** 需要充分利用如 **AVX2** 等专用指令，并尽可能保持核心频率在最高水平。
* **高内存带宽和容量：** 大规模数据集的加载和处理对内存速度和 **NUMA** 拓扑十分敏感。
* **高效数据传输：** 快速的互连（例如 **PCIe**）对于将计算任务卸载到 GPU 等加速器至关重要。

考虑到这些特点，以下是需要重点关注的 BIOS 设置。


### 关键 BIOS 设置详解

#### 1. NUMA 配置（NPS - 每插槽 NUMA 域）

**目标：** 优化数据局部性和内存带宽。AI 工作负载对数据访问模式极为敏感。

* **建议：** 对于大多数 AI 工作负载，推荐将 **NPS** 设置为 **2** 或 **4**。
    * **NPS=2**：每个处理器插槽创建两个 NUMA 域。这能有效利用内存带宽，是兼顾性能和通用性的优秀选择。
    * **NPS=4**：每个处理器插槽创建四个 NUMA 域。该设置适合高度并行，且每个 NUMA 节点的数据集较小的工作负载。通过将进程显式绑定到特定 NUMA 节点，可以实现极致的数据局部性。

* **如何检查：** 在 Linux 系统中，您可以使用 `numactl --hardware` 或 `hwloc-ls` 命令来验证更改后的 NUMA 拓扑。

{{< admonition >}}
实践下来发现：NPS=1，对于推理任务是更优的选择。
{{< /admonition >}}

#### 2. 核心性能和功耗设置

**目标：** 确保 CPU 核心始终以最高频率稳定运行，并最小化延迟。

* **性能模式：** 将 **“Determinism Slider”** 或类似的性能模式选项设置为 **“Performance”** 或 **“Max Performance”**。这会优先考虑性能而非能效，让核心时钟保持在高位。
* **C-states：** 将 **“Global C-state Control”** 或 **“C-States”** 设置为**“Disabled”**。C-states 是 CPU 的节电模式，虽然能节省能源，但在核心需要从低功耗状态唤醒时会引入延迟，从而影响 AI 训练的一致性。
* **P-states：** 寻找 **“P-states Control”**、**“DF P-states”** 或 **“APBDIS”** 等设置。**禁用** P-states 或确保处理器锁定在其最高性能状态（P0）。这能防止 CPU 频率随负载动态调整，从而保证最高时钟速度。

#### 3. 内存设置

**目标：** 最大化内存带宽，同时降低延迟。

* **内存速度：** 将内存配置为在主板和 DIMM 所支持的**最优延迟**下运行，例如 **DDR4-2933 MT/s**，并不推荐 3200 MT/s，因为7002系列的Infinity Fabric时钟为2933，使内存时钟与Infinity Fabric时钟保持同步对于低延迟是更优的选择。
* **内存交错：** 确保内存交错配置能提供**最佳带宽**。通常，为每个插槽填满所有 **8 个内存通道**能获得最大带宽。具体设置请参考您系统的官方文档。

#### 4. PCIe 和 IOMMU

**目标：** 确保 GPU 和其他加速器之间能实现最佳通信。

* **PCIe 代数：** 将用于 GPU 的 PCIe 插槽设置为其**支持的最高代数和链路宽度**（例如 **PCIe Gen4 x16**）。
* **Above 4G Decoding：** 务必**启用**该选项。这对于拥有大量 GPU 显存的系统至关重要，能让系统正确识别和映射超过 4GB 的 I/O 内存。
* **IOMMU：** 如果您使用虚拟化或需要设备直通的容器技术，请保持 **IOMMU** **启用**。对于裸机 AI 训练，禁用该选项可能会带来微不足道的性能提升，但为了系统稳定性和日后的灵活性，通常建议保持启用。


### 推荐的基线 BIOS 设置

下表提供了一套经过验证的，适用于 AI 工作负载的 BIOS **基线设置**。

#### ACPI Settings

{{< image src="./images/ACPI Settings.png" alt="ACPI Settings" caption="ACPI Settings" width="60%" >}}

| 项目                                  | 设置值      |
| :------------------------------------ | :---------- |
| High Precision Event Timer            | [Disabled]  |
| NUMA Nodes Per Socket                 | [NPS1]      |
| ACPI SRAT L3 Cache As NUMA Domain     | [Enabled]   |



#### North Bridge Configuration

{{< image src="./images/North Bridge Configuration.png" alt="North Bridge Configuration" caption="North Bridge Configuration" width="60%" >}}

| 项目                              | 设置值     |
| :-------------------------------- | :--------- |
| Determinism Control               | [Manual]   |
| Determinism Slider                | [Performance] |
| cTDP Control                      | [Manual]   |
| cTDP                              | 220        |
| IOMMU                             | [Enabled]  |
| Package Power Limit Control       | [Manual]   |
| Package Power Limit               | 220        |
| APBDIS                            | [0]        |
| DF Cstates                        | [Disabled] |



#### Memory Configuration

{{< image src="./images/Memory Configuration.png" alt="Memory Configuration" caption="Memory Configuration" width="60%" >}}

| 项目            | 设置值   |
| :-------------- | :------- |
| Memory Clock    | [2933MHz] |
| TSME            | [Disabled] |

#### CPU Configuration

{{< image src="./images/CPU Configuration.png" alt="CPU Configuration" caption="CPU Configuration" width="60%" >}}

| 项目                  | 设置值     |
| :-------------------- | :--------- |
| Global C-state Control | [Disabled] |

#### PCI Devices Common Settings

{{< image src="./images/PCI Devices Common Settings.png" alt="PCI Devices Common Settings" caption="PCI Devices Common Settings" width="60%" >}}

| 项目                 | 设置值     |
| :------------------- | :--------- |
| Above 4G Decoding    | [Enabled]  |
| SR-IOV Support       | [Enabled]  |
| VGA Priority         | [Offboard] |


### 通用调优原则和后续步骤

* **基准测试和迭代：** BIOS 调优是一个持续的迭代过程。每次更改后，都应运行特定的 AI 工作负载基准测试，衡量性能（如吞吐量、训练速度）并根据结果进行调整。
* **操作系统调优：** 别忘了，BIOS 只是优化过程的一部分。操作系统级别的优化同样关键，例如使用 **`numactl`** 等工具进行 CPU 亲和性绑定、调整内核参数或禁用不必要的后台服务。
* **软件栈优化：** 确保您的 AI 框架（如 TensorFlow, PyTorch）、底层库（如 cuDNN, MKL, ROCm）和驱动程序（NVIDIA, AMD）都已更新到最新版本并进行了相应的优化配置。

通过遵循上述指南，您可以显著提升 AMD EPYC 7002 系统在 AI 工作负载下的性能和效率。

---

> 作者: pr_zutto  
> URL: https://przutto.github.io/ai-server/a1df1f9/  

