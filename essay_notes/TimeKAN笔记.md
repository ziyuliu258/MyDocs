## Abstract

本文受近期 **Kolmogorov-Arnold Network（KAN）** 的灵活性启发，提出了一种基于 KAN 的**频率分解学习架构（TimeKAN）**，用于应对由多频率混合引发的复杂预测挑战。

具体来说，TimeKAN 主要由三个部分组成
1. **级联频率分解（Cascaded Frequency Decomposition, CFD）模块**：采用自底向上的级联方式，获得每个频率带的序列表示；
2. **多阶 KAN 表征学习（Multi-order KAN Representation Learning, M-KAN）模块**：利用 KAN 的高灵活性，在各频率带中学习并表示特定的时间模式；
3. **频率混合（Frequency Mixing）模块**：将不同频率带重新组合为原始格式。

受益于 KAN 在函数拟合上的强大灵活性，TimeKAN 能够根据不同频率模式的复杂性，动态调整 KAN 的阶数，从而提升不同频率成分的建模能力。

-----
## Introduction

现实世界中的多变量时间序列往往具有**非平稳性（non-stationarity）** 和 **复杂的多样模式（diverse patterns）**。这些交织的模式使得时间序列内部的依赖关系变得复杂，从而难以准确建立历史观测与未来目标之间的联系。为了应对时间序列中的复杂时序模式，越来越多的研究开始利用**先验知识（prior knowledge）** 将时间序列分解为若干更简单的子成分，以辅助预测任务。
不同频率成分的混合极大增加了预测的难度。受到上述分解思想的启发，我们希望设计一种**频率分解框架（frequency decomposition framework）**，能够将时间序列中的不同频率成分进行解耦，并分别学习每个频率对应的时间模式。然而，这又引出了新的问题：

> 不同频率的模式信息密度不同，若使用统一的建模方式，会造成对部分频率特征的错误表征，导致预测性能下降。

幸运的是，近年来出现的一种全新神经网络结构——**Kolmogorov–Arnold Network（KAN）**（Liu et al., 2024c），以其卓越的数据拟合能力与灵活性引起了广泛关注，被认为是传统 **MLP** 的潜在替代方案。相比于 MLP，KAN 拥有**可选核函数（optional kernels）** 与**可调阶数（order adjustable）** ，从而能够灵活控制模型的拟合能力。这启发我们探索使用**多阶 KAN（Multi-order KAN）** 来建模不同频率下的时间模式，使模型能够根据频率复杂度动态调整其表达能力，从而更精准地捕获多频率信息。

基于以上思考，本文提出了 **TimeKAN** ——一种**基于 KAN 的频率分解学习架构（Frequency Decomposition Learning Architecture）**，专门用于解决由多频率混合引起的复杂预测问题。
TimeKAN 的核心思想是遵循“**分解–学习–混合（Decomposition–Learning–Mixing）**”的建模流程：
1. 通过**滑动平均（moving average）** 逐步去除高频成分；
2. 使用**级联频率分解模块（Cascaded Frequency Decomposition, CFD）** 以自底向上的方式提取各个频带的序列表示；
3. 设计**多阶 KAN 表征学习模块（Multi-order KAN Representation Learning, M-KAN）**，利用 KAN 的灵活性在不同频带内学习对应的时间模式；
4. 最后通过**频率混合模块（Frequency Mixing）** 将各频带重新组合为原始序列格式，使整个过程可重复执行，从而实现对不同频率模式的精确建模。
最终，高层序列将通过一个简单的线性映射映射到预测结果上。

## Related Works

### **2.1 Kolmogorov–Arnold Network（KAN）**

**Kolmogorov–Arnold 表示定理**指出：

> 任何多变量连续函数都可以表示为若干**单变量函数**与**加法运算**的组合。

基于这一数学原理，**Kolmogorov–Arnold Network（KAN）**（Liu et al., 2024c）被提出，作为一种替代传统多层感知机（MLP）的新型神经网络架构。与传统的 MLP 不同，后者在节点上使用固定的激活函数，而 **KAN 在“边”上引入了可学习的单变量函数**，从而使得模型能够灵活地调整非线性映射。由于其极高的灵活性与自适应能力，KAN 被认为是 MLP 的一个有前景的替代方案。
最初的 KAN 使用**样条函数（spline functions）** 作为可学习的激活函数，但由于样条函数计算复杂、递归结构深，原始版本在速度与可扩展性方面表现不理想。  因此，后续研究尝试用更简单的基函数取代样条函数，从而提升效率。例如：

- **ChebyshevKAN（SS, 2024）**：使用 **Chebyshev 多项式**来参数化可学习函数；
- **FastKAN（Li, 2024）**：用高斯径向基函数（Gaussian RBF）近似三阶 B 样条函数，以实现更快计算。
>在SVM算法中也出现了这样的RBF函数，要复习。

目前，KAN 已被广泛应用于多个领域，作为 MLP 的替代组件：

- **卷积式 KAN（Convolutional KAN）**（Bodner et al., 2024）：将传统卷积网络中的线性权重矩阵替换为可学习的函数矩阵，从而提升卷积的表达能力；
- **U-KAN（Li et al., 2024）**：将 KAN 层集成到 U-Net 架构中，在医学图像分割任务中表现出色，兼具高精度与高效率；
- **PIKAN / PINN**（Shukla et al., 2024；Wang et al., 2024b）：结合物理约束，将 KAN 应用于“物理信息神经网络（Physics-Informed Neural Network）”中，用于科学计算和物理建模任务。

本论文首次将 **KAN 引入时间序列预测（TSF）领域**，展示了其在时间依赖建模和特征表示中的强大潜力。

---
### **2.2 时间序列预测（Time Series Forecasting）**

传统的时间序列预测方法（如 **ARIMA**（Zhang, 2003））虽然具有较强的可解释性，但在复杂序列上的预测精度通常不足。  
近年来，深度学习方法主导了 TSF 领域，主要分为以下三类：
#### （1）CNN-based 方法

这类方法通过在时间维度上进行卷积操作来提取时序模式。  
例如：

- **MICN（Wang et al., 2023）** 和 **TimesNet（Wu et al., 2023）** 调整感受野（receptive field），在捕捉短期与长期依赖关系上取得平衡；
- **ModernTCN（donghao & wang xue, 2024）** 采用大卷积核以同时捕捉跨时间与跨变量的依赖关系。
#### （2）Transformer-based 方法

相比于卷积网络的局部感受野，Transformer 通过全局自注意力机制具备更强的长依赖建模能力，因此成为现代时间序列预测的核心模型。
代表性工作包括：

- **Informer（Zhou et al., 2021）**：改进了标准 Transformer 的注意力结构，提升了长序列预测的效率；
- **PatchTST（Nie et al., 2023）**：将时间序列划分为多个 patch 输入 Transformer，大幅减少计算开销；
- **iTransformer（Liu et al., 2024b）**：将每个变量视为独立 token，以建模多变量序列的跨变量依赖。

不过，这类方法仍然面临参数量庞大、显存占用高的问题。

#### （3）MLP-based 方法

近期研究发现，通过合理设计结构并融入时间序列的先验知识，**简单的 MLP** 也能超越复杂的 Transformer 模型。

代表性模型包括：

- **DLinear（Zeng et al., 2023）**：利用滑动平均进行趋势–季节分解；
- **FITS（Xu et al., 2024b）**：在频域执行线性变换；
- **TimeMixer（Wang et al., 2024a）**：在不同尺度上通过 MLP 实现信息交互。

这些模型在精度和计算效率上都表现出色。  
与上述方法不同，本文首次将 **KAN 引入时间序列预测任务**，以实现更高精度的表示学习，同时构建一种全新的 **分解–学习–混合（Decomposition–Learning–Mixing）** 架构，以充分发挥 KAN 的潜力。

---

### **2.3 时间序列分解（Time Series Decomposition）**

现实世界的时间序列通常包含多种潜在模式。为了更好地利用这些模式特征，研究者们倾向于将原始序列分解为多个子成分，包括**趋势–季节分解**、**多尺度分解**和**多周期分解**等。

典型代表包括：

- **DLinear（Zeng et al., 2023）**：使用滑动平均（moving average）分离趋势与季节性成分；
- **SCINet（Liu et al., 2022）**：通过分层下采样树（hierarchical downsampling tree）在多时间尺度上迭代提取与交换信息；
- **TimeMixer（Wang et al., 2024a）**：遵循“由细到粗（fine-to-coarse）”原则，将序列分解为多尺度结构，并进一步划分为季节性与周期性部分；
- **TimesNet（Wu et al., 2023）** 与 **PDF（Dai et al., 2024）**：利用傅里叶周期分析（Fourier Periodic Analysis）将序列分解为多个子周期，以分别建模不同周期的内部规律。

受到这些工作的启发，本文提出了一种**新型的分解–学习–混合架构（Decomposition–Learning–Mixing Architecture）**，从**多频率（multi-frequency）** 的角度重新审视时间序列建模问题，从而更精确地捕获序列中的复杂模式。