---
title: "SIGGRAPH 2026 清华三篇获奖论文精读：蒙特卡洛估计中的相关性艺术"
date: 2026-05-28
draft: false
tags: ["图形学", "蒙特卡洛", "渲染", "PDE求解", "SIGGRAPH", "论文研读"]
categories: ["论文研读"]
description: "SIGGRAPH 2026 最佳论文提名中，清华大学三篇工作有一条共同主线：如何通过挖掘样本间的相关性，在极有限的采样预算下实现大幅降方差。本文精读这三篇论文的核心观点与推导过程。"
showToc: true
TocOpen: true
math: true
---

> 本文基于微信公众号"图形学与人工智能"对 SIGGRAPH 2026 清华获奖论文的介绍，结合论文原理进行深度精读与分析。
>
> 原文链接：[SIGGRAPH 2026 论文奖项揭晓、清华大学三篇论文获最佳论文提名](https://mp.weixin.qq.com/s/TRb5qC0BUqLAuubEl2E6nA)

## 一、背景：一条隐藏的方法主线

SIGGRAPH 2026 共评出 5 篇最佳论文（Best Paper）和 10 篇最佳论文提名（Honorable Mention）。在 10 篇提名中，清华大学包揽三席——计算机系徐昆课题组贡献两篇，软件学院徐枫课题组贡献一篇。

三篇论文看起来方向各异：

- **论文一**：蒙特卡洛 PDE 求解（偏微分方程数值解）
- **论文二**：实时路径追踪渲染降噪
- **论文三**：可微体渲染中的逆向重建

但它们共享一条深刻的方法主线：

> **在样本预算极为有限的情况下，如何通过挖掘样本之间天然存在的相关性，以接近零额外成本的代价显著降低蒙特卡洛估计的方差？**

理解这条主线，需要先回顾蒙特卡洛估计的基本框架。

---

## 二、方法前置：蒙特卡洛估计与方差

### 2.1 蒙特卡洛估计的基本形式

对于积分 $I = \int_\Omega f(x)\, dx$，蒙特卡洛估计器为：

$$\hat{I} = \frac{1}{N} \sum_{i=1}^N \frac{f(x_i)}{p(x_i)}$$

其中 $x_i \sim p(x)$ 是从概率密度 $p$ 中独立采样的样本。估计器是无偏的，但方差为：

$$\mathrm{Var}[\hat{I}] = \frac{1}{N} \mathrm{Var}\!\left[\frac{f(x)}{p(x)}\right]$$

**降方差的两条路**：
1. 增加样本数 $N$（成本线性增长）
2. 在不增加 $N$ 的前提下降低单样本方差（"免费的午餐"）

三篇论文的核心贡献，正是在第二条路上各自找到了独特的切入点。

### 2.2 控制变量（Control Variates）原理

控制变量是一种经典降方差技术。若已知某个函数 $c(x)$ 满足 $\mathbb{E}[c(x)] = \mu_c$，则构造新的估计量：

$$\hat{I}_\alpha = \hat{I} - \alpha(\hat{c} - \mu_c)$$

选取最优系数 $\alpha^* = \frac{\mathrm{Cov}[\hat{I}, \hat{c}]}{\mathrm{Var}[\hat{c}]}$，新估计量的方差为：

$$\mathrm{Var}[\hat{I}_\alpha] = \mathrm{Var}[\hat{I}] \cdot (1 - \rho^2)$$

其中 $\rho$ 是 $\hat{I}$ 与 $\hat{c}$ 之间的相关系数。**相关性越强，方差降幅越大**。这正是论文一和论文二的方法核心。

---

## 三、论文一：梯度域重建用于蒙特卡洛 PDE 求解

**论文标题**：*Gradient Domain Reconstruction for Monte Carlo PDE Solvers*

**作者**：吴佳启、胡雪君、赵爽、徐昆（清华大学 + UIUC）

**项目主页**：https://jiaoplusjuan.github.io/GDMCPDE.html

### 3.1 问题背景：Walk on Spheres 的局限

无网格蒙特卡洛 PDE 求解器，尤其是 Walk on Spheres (WoS) 和 Walk on Stars (WoSt)，是近年图形学领域解偏微分方程的前沿方法。以泊松方程为例：

$$\Delta u(x) = f(x), \quad x \in \Omega$$
$$u(x) = g(x), \quad x \in \partial\Omega$$

WoS 方法的核心思想是：对于任意查询点 $x$，在以 $x$ 为中心的最大无障碍球 $B(x)$ 内，通过格林函数的均值性质，将 $u(x)$ 的估计转化为球面上的随机游走：

$$u(x) = \mathbb{E}[u(y)] + \int_B G_B(x, z) f(z)\, dz$$

其中 $y$ 是球面上的均匀随机点，$G_B$ 是球内格林函数。重复直到游走抵达边界 $\partial\Omega$，用 $g$ 的边界值作为终止估计。

**核心问题**：WoS 对每个查询点 $x$ **独立**采样。若要估计一组相邻查询点 $\{x_1, x_2, \ldots, x_K\}$ 上的解（如渲染中的像素网格），每个点都各自跑一套独立的随机游走，天然的空间相关性完全被忽视，低样本下噪声极大。

### 3.2 核心思路：借鉴梯度域渲染

梯度域渲染是离线渲染中的经典降方差技术：不仅估计每个像素的辐射度值，还额外估计相邻像素之间的**辐射度差分**（梯度），再通过泊松重建将值估计和梯度估计融合，得到整体方差更低的结果。

论文将这一思路迁移到 WoS/WoSt 框架下：

**关键洞察**：对两个相邻查询点 $x$ 和 $x'$，如果 $x'$ 位于以 $x$ 为中心的内切球 $B(x)$ 内，则 $x'$ 可以被视为同一个球内的**偏心点**。利用球内的**偏心格林函数**（Off-center Green's Function），可以从 $x$ 的游走样本中**额外免费**地估计出 $u(x) - u(x')$ 的差分，而无需对 $x'$ 单独采样。

具体而言，对于球 $B$ 内的偏心点 $x'$，格林函数满足：

$$G_B(x', z) = G_B(x, z) + \delta G_B(x, x', z)$$

其中 $\delta G_B$ 是偏心修正项，可以解析计算。于是差分估计为：

$$\widehat{u(x) - u(x')} = \int_B \delta G_B(x, x', z) f(z)\, dz$$

这个积分可以复用 $x$ 的游走首步内切球及其样本来估计，**零额外游走代价**。

### 3.3 重建框架：广义无偏重建

将所有查询点的值估计 $\{\hat{u}(x_k)\}$ 和差分估计 $\{\widehat{u(x_i) - u(x_j)}\}$ 组合，构成超定线性系统：

$$\mathbf{A} \mathbf{u} \approx \mathbf{b}$$

其中 $\mathbf{A}$ 是由值约束和差分约束构成的系数矩阵，$\mathbf{b}$ 是对应的蒙特卡洛估计值向量。求解这个**稀疏线性系统**（标准最小二乘），即可得到融合了相关信息的最终解向量 $\mathbf{u}^*$。

整个框架的关键性质：
- **无偏性**：偏心格林函数估计量本身是无偏的，融合后不引入 bias
- **方差下降**：差分估计与值估计之间存在强正相关，重建后方差显著降低
- **高梯度区域收益最大**：解变化剧烈的区域，相邻点之间的相关性最强

### 3.4 实验结果

在等时间（相同采样成本）对比中：
- 面向全局查询点网格，误差显著低于 WoS/WoSt 基线
- 面向局部 ROI（高梯度、复杂边界区域），优势更加明显
- 自然推广到非均匀采样点、切片查询等场景

**论文亮点**：第一作者吴佳启是清华计算机系**大四本科生**，将渲染领域的梯度域思想迁移到 PDE 求解领域，展示出跨方向迁移的研究视野。

---

## 四、论文二：ReSTIR 时空控制变量

**论文标题**：*Spatio-Temporal Control Variates with ReSTIR for Real-Time Rendering*

**作者**：时中、吴存浩、巫立凡、徐昆（清华大学 + NVIDIA）

**项目主页**：https://hercier.github.io/restcv/

### 4.1 问题背景：ReSTIR 的颜色噪声困境

实时路径追踪（real-time path tracing）是现代游戏渲染的前沿技术（如 NVIDIA RTX 系列）。由于实时约束，每像素只能采集极少量（通常 1-4 个）光路样本，结果噪声极大。

**ReSTIR**（Reservoir-based Spatio-Temporal Importance Resampling）是当前最主流的实时降噪框架，核心机制是：

1. **蓄水池采样（Reservoir Sampling）**：用流式算法维护一个代表性"蓄水池"样本
2. **时间复用**：将上一帧的样本复用到当前帧
3. **空间复用**：将相邻像素的样本复用

每个像素最终持有一个按**标量目标函数**（通常是亮度 luminance）选出的单一代表样本。这样能很好地拟合"高亮度区域"，但存在一个根本性缺陷：

> **颜色信息是三维向量（R,G,B），但 ReSTIR 只按标量亮度选样本。**

以红绿两色光源照射白色平面为例：正确结果是黄色（R+G 均等），但 ReSTIR 的单代表样本每次只能"选红"或"选绿"，导致像素颜色在红绿之间跳变，表现为明显的色彩噪声（color noise）。

### 4.2 图像空间控制变量（ICV）的先验工作

图像空间控制变量（Image-space Control Variates, ICV）是一种基于相邻像素颜色相关性的降方差技术：

设相邻像素 $i$ 和 $j$ 的真实辐射度分别为 $L_i$ 和 $L_j$，若能估计它们之间的差分 $L_i - L_j$（方差较低），则对像素 $i$ 的估计可以改写为：

$$\hat{L}_i^{\mathrm{ICV}} = \hat{L}_j + \widehat{(L_i - L_j)}$$

由于 $\hat{L}_j$（来自邻居）和 $\widehat{(L_i - L_j)}$（差分估计）之间的相关性强，组合后方差显著低于直接估计 $\hat{L}_i$。

**但 ICV 的问题**：差分估计需要两个像素用**相同的样本路径**（shift mapping，转移映射）来计算，这需要在独立采样框架下额外生成样本，开销较高。

### 4.3 核心贡献：ReSTCV = ICV × ReSTIR 的统一

论文的关键洞察是：

> **ReSTIR 在时空复用过程中已经产生了大量"候选样本"，这些候选样本恰好可以用来做 ICV 的差分估计，而不需要任何额外采样。**

具体来说，ReSTIR 在空间复用阶段，已经从相邻像素 $j$ 的蓄水池中"借"来了样本 $\bar{x}_j$，并通过**转移映射**（shift mapping）将其变换到像素 $i$ 的路径空间上，评估其贡献。这个转移映射的过程，恰好是 ICV 估计差分 $L_i - L_j$ 所需要的。

论文将 ICV 扩展为**时空控制变量**（Spatio-Temporal Control Variates, STCV），使其结构与 ReSTIR 完全吻合：

- **时间 STCV**：利用时间复用中的转移映射，估计当前帧与上一帧对应像素的颜色差分
- **空间 STCV**：利用空间复用中的转移映射，估计当前像素与相邻像素的颜色差分

最终估计器为：

$$\hat{L}_i^{\mathrm{ReSTCV}} = \hat{L}_i^{\mathrm{ReSTIR}} - \sum_{j \in \mathcal{N}(i)} \alpha_{ij} \left(\hat{L}_j^{\mathrm{shift}} - \hat{L}_j\right)$$

其中 $\hat{L}_j^{\mathrm{shift}}$ 是将像素 $j$ 的样本映射到像素 $i$ 后评估的贡献，该量在 ReSTIR 的正常执行中已经被计算过，**真正零额外开销**。

### 4.4 数学统一视角

论文不只是一个工程技巧，更建立了一个数学上的统一框架：

**ReSTIR、ICV、梯度域渲染**三者都是"基于转移映射的方差缩减方法"的特例。转移映射 $T: \Omega_j \to \Omega_i$ 将像素 $j$ 的光路样本映射到像素 $i$ 的路径空间，Jacobian 修正保证无偏性。三种方法的区别仅在于：

| 方法 | 转移映射用途 |
|------|------------|
| ReSTIR | 用于重采样权重（标量亮度） |
| ICV | 用于差分估计（颜色向量） |
| 梯度域渲染 | 用于显式梯度估计 + 泊松重建 |

ReSTCV 将三者融合：在 ReSTIR 的重采样结构内，同时复用转移映射做颜色差分估计，实现了"1+1+1=1"的优雅合并。

### 4.5 实验结果

- 复杂间接光照场景：色彩方差显著降低，细节更清晰
- 多色光源直接光照：有效消除红/绿颜色跳变
- 运行时间开销：与原始 ReSTIR 基本持平（< 5% 额外开销）

---

## 五、论文三：可微体渲染中的样本匹配

**论文标题**：*Sample Matching for Joint Extinction Gradient Estimation in Differentiable Volume Rendering*

**作者**：余睿涵、王雨晨、凌静旺、徐枫、赵爽（东京大学 + 清华大学 + UCI + UIUC）

### 5.1 问题背景：体积逆渲染与梯度方差

**可微体渲染**（Differentiable Volume Rendering）是通过多视角图像反向优化重建三维体积场景的范式，支持 voxel grid、NeRF、3DGS 等多种表达形式。基于多次反射路径追踪的重建能恢复准确的材质与几何。

体积渲染方程（Volume Rendering Equation）沿光线 $\mathbf{r}(t) = \mathbf{o} + t\mathbf{d}$ 的辐射度为：

$$L(\mathbf{o}, \mathbf{d}) = \int_0^\infty T(t) \cdot \sigma(t) \cdot L_s(\mathbf{r}(t), \mathbf{d})\, dt$$

其中透射率 $T(t) = \exp\!\left(-\int_0^t \sigma(s)\, ds\right)$，$\sigma$ 为消光系数（extinction coefficient）。

逆渲染需要计算损失函数 $\mathcal{L}$ 对消光系数 $\sigma$ 的梯度：

$$\frac{\partial L}{\partial \sigma(\mathbf{x})} = \underbrace{\frac{\partial}{\partial \sigma} \int T \cdot \sigma \cdot L_s\, dt}_{\text{散射项（scattering）}} + \underbrace{\frac{\partial}{\partial \sigma} \int T(\cdot) \cdot (\cdot)\, dt}_{\text{透射项（transmittance）}}$$

对消光系数求导，产生**两个符号相反的分量**：

- **散射项**（scattering component）：在路径顶点 $\mathbf{x}$ 处取值，反映密度增大导致散射辐射增强（**正贡献**）
- **透射项**（transmittance component）：沿光线段积分，反映密度增大导致透射衰减（**负贡献**）

二者的物理含义完全对立——在某点增大密度，既增强了该点的散射辐射，又衰减了穿过该点的背景辐射。

**关键洞察**：这种对立意味着两项之间存在**天然的负协方差**：

$$\mathrm{Cov}[\hat{g}_\mathrm{scat}, \hat{g}_\mathrm{trans}] < 0$$

若能同时估计两项（即令它们负相关），则总梯度方差为：

$$\mathrm{Var}[\hat{g}_\mathrm{scat} + \hat{g}_\mathrm{trans}] = \mathrm{Var}[\hat{g}_\mathrm{scat}] + \mathrm{Var}[\hat{g}_\mathrm{trans}] + 2\mathrm{Cov}[\hat{g}_\mathrm{scat}, \hat{g}_\mathrm{trans}]$$

负协方差项使总方差**低于**两者单独方差之和——这正是"免费的午餐"。

### 5.2 现有方法的缺陷：协方差被强行归零

然而现有方法恰恰破坏了这一负相关性：

- **自由飞行采样**（Free-flight Sampling）：散射项和透射项在几何上**不同的位置**评估，相关性为零
- **微分比值追踪 DRT**（Differential Ratio Tracking，当前 SOTA）：对两项进行**独立抽样**，协方差强制为零

这等价于人为抛弃了梯度中天然存在的方差缩减结构。

### 5.3 样本匹配（Sample Matching）原理

论文的核心贡献是证明：存在一种**等价重写**，将散射项与透射项改写到**同一积分域**，从而在同一采样位置联合估计两者的梯度。

**推导过程**（简化版）：

对某段光线区间 $[a, b]$，透射率 $T(a, b) = \exp\!\left(-\int_a^b \sigma(s)\, ds\right)$，其对 $\sigma(x)$ 的导数（透射项）为：

$$\frac{\partial T(a,b)}{\partial \sigma(x)} = -T(a, b), \quad x \in [a, b]$$

而散射贡献（散射项）对应路径顶点 $x$ 处的 $\sigma(x) \cdot L_s(x)$。

利用 **Fubini 定理**交换积分顺序，并将透射率写成其积分定义 $T(a,t) = 1 - \int_a^t \sigma(s) T(a,s)\, ds$，可以将透射项的沿段积分改写为对采样点 $t$ 的期望：

$$\int_a^b \frac{\partial T(a,b)}{\partial \sigma(x)} L_\mathrm{bg}\, dt = \mathbb{E}_{t \sim T\text{-sampling}}\!\left[-L_\mathrm{bg}(t)\right]$$

其中 $t$ 与散射项中的路径顶点处在**同一采样域**上。在此新表达下，散射项和透射项被强迫在同一点 $t$ 处联合评估，负协方差得以充分利用。

**实现层面**：

- 复用现有路径采样（无额外路径）
- 每段采用 multi-sample per segment 均摊入射散射评估的开销
- 在 Mitsuba 3 等现代可微渲染框架上，**极少量代码改动**即可实现

### 5.4 实验结果

在 8 个包含环境光照与高分辨率异质体素网格的场景上：

- **体素级梯度方差**：相比 SOTA 方法 DRT，最高降低 **80%**
- **空旷区域效果最显著**：密度接近零时，"漂浮噪点"是逆渲染的顽疾，样本匹配可有效消除
- **重建质量**：在 surface-like 和 volumetric 物体上，重建结果噪声均显著低于 DRT 基线
- **逆渲染整体质量**：全面超越 DRT 基线

---

## 六、三篇论文的统一视角

### 6.1 相关性的三种形态

| 论文 | 相关性来源 | 利用方式 | 收益 |
|------|----------|---------|------|
| 梯度域 PDE（论文一） | 相邻查询点的空间相关性 | 偏心格林函数估计差分 + 联合重建 | 相同采样预算下更低误差 |
| ReSTCV（论文二） | 相邻像素的颜色相关性 | 复用 ReSTIR 转移映射估计颜色差分 | 消除色彩噪声，零额外开销 |
| 样本匹配（论文三） | 散射项与透射项的负相关性 | Fubini 变换到同一积分域联合估计 | 梯度方差最高降低 80% |

### 6.2 降方差的设计哲学

三篇工作体现了一个共同的设计原则：

> **不要生造新的相关性，而是去发现并利用问题结构中天然存在但长期被忽略的相关性。**

- 论文一的相邻查询点相关性，天然存在于 PDE 解的光滑性中
- 论文二的像素颜色相关性，天然存在于图像的空间连续性中
- 论文三的散射-透射负相关，天然存在于辐射传输方程的数学结构中

它们的共同代价是：接近零的额外采样成本（多一些矩阵运算或重写积分域），换来显著的方差缩减。这是图形学中"好的降方差技术"的典型形态。

### 6.3 方法迁移的跨域视角

论文一将**渲染中的梯度域重建**迁移到 PDE 求解领域；论文二将**离线渲染中的 ICV** 迁移到实时渲染的 ReSTIR 框架。这种跨子领域的方法迁移，是图形学研究中值得关注的研究范式——当一个领域的方法在其他领域的类似结构上被应用时，往往能产生优雅且高效的新方法。

---

## 七、总结

SIGGRAPH 2026 清华三篇提名奖论文，表面上分属蒙特卡洛 PDE、实时渲染、可微体渲染三个子方向，实则共享一条深刻的方法论主线：**发现并系统利用问题结构中天然存在的样本相关性，以极低的额外代价实现大幅方差缩减**。

三篇工作的贡献不止于具体技术：
- 论文一建立了 WoS/WoSt 框架与梯度域重建的连接
- 论文二在数学层面统一了 ReSTIR、ICV 与梯度域渲染三种看似不同的方法
- 论文三发现了辐射传输方程中长期被忽略的负相关结构，为可微体渲染梯度估计器设计提供了新思路

这三篇工作提醒我们：在蒙特卡洛估计的世界里，**最好的样本不一定是最多的，而是彼此之间关联最深的**。

---

## 参考文献

1. ACM SIGGRAPH. *SIGGRAPH 2026 Technical Papers Awards: Best Papers, Honorable Mentions, and Test-of-Time.* SIGGRAPH Blog, May 2026.
2. Jiaqi Wu, Xuejun Hu, Shuang Zhao, Kun Xu. *Gradient Domain Reconstruction for Monte Carlo PDE Solvers.* ACM Transactions on Graphics (SIGGRAPH 2026).
3. Zhong Shi, Cunhao Wu, Lifan Wu, Kun Xu. *Spatio-Temporal Control Variates with ReSTIR for Real-Time Rendering.* ACM SIGGRAPH 2026 Conference Paper.
4. Ruihan Yu, Yu-Chen Wang, Jingwang Ling, Feng Xu, Shuang Zhao. *Sample Matching for Joint Extinction Gradient Estimation in Differentiable Volume Rendering.* ACM Transactions on Graphics (SIGGRAPH 2026).
5. Cheng Zhang, Lifan Wu, Changxi Zheng, Ioannis Gkioulekas, Ravi Ramamoorthi, Shuang Zhao. *A differential theory of radiative transfer.* ACM Transactions on Graphics, 2019.
6. Merlin Nimier-David, Thomas Müller, Alexander Keller, Wenzel Jakob. *Unbiased inverse volume rendering with differential trackers.* ACM Transactions on Graphics, 2022.
