# **2025年亚太地区大学生数学建模竞赛**

## 赛题 B

### **辐射制冷技术的建模与优化**

全球工业化和人口增长导致能源消耗持续增加。尽管太阳能、风能和核能等清洁能源的供应比例有所上升，但石油和其他液体燃料仍然是世界上最大的能源供应来源。这导致了城市热岛效应和温室效应的加剧。

根据斯特藩-玻尔兹曼定律（Stefan-Boltzmann Law），温度较高的物体比温度较低的物体辐射更多的功率。在地球表面平均温度为 15∘C 的条件下，地表物体的大部分热辐射发生在红外光谱范围内。因此，科学家们提出了一种被称为“被动日间辐射制冷”（Passive Daytime Radiative Cooling, PDRC）的技术。该技术基于以下原理：红外辐射在 **8−13 μm** “大气透明窗口”具有**高透射率**，以及**由于温差发生的热辐射能量交换**。在不消耗任何能源的情况下，地表物体可以直接将其热辐射能量发射到寒冷的外太空中，从而自发降低其温度，甚至可能低于环境温度。

![image-20251120102742335](/home/ziyu/.config/Typora/typora-user-images/image-20251120102742335.png)

*[图1：辐射制冷技术（左图示意了太阳辐射反射与大气窗口的热辐射；右图展示了太阳光谱、大气窗口以及发射率/吸收率的关系）]*

与其他制冷技术相比，辐射制冷不需要能量输入，同时还能减少整体能源消耗，形成节能的良性循环。这对于实现可再生能源发展和碳中和目标具有重要意义。该技术在节能建筑、太阳能电池、个人热管理、电厂冷凝器以及发电和集水等领域显示出巨大的应用潜力。

然而，辐射制冷技术在通往大规模实际应用的道路上仍面临诸多挑战。这些挑战主要在于需要优化材料的光谱发射特性，以及复杂的制造工艺和高昂的生产成本。目前的研究表明，聚二甲基硅氧烷（PDMS）薄膜是一种在可见光谱中具有超高透射率，且在“大气窗口”波段具有高发射率的材料，非常适合应用于辐射制冷技术。

有关材料折射率的信息可以在相关网站上找到（例如：https://refractiveindex.info/）。

#### **问题：**

1. 请建立一个合适的数学模型，研究不同薄膜厚度下 PDMS 薄膜的发射率随波长变化的关系。
2. 请建立一个合适的评价模型，评估不同厚度 PDMS 薄膜的辐射制冷性能。基于你的发现，为辐射制冷技术的发展和应用提供一些建议。
3. 请选择合适的材料与 PDMS 结合形成多层膜结构。结合优化设计算法，对多层结构中每一层的材料选择和层厚度进行优化。
4. 请对辐射制冷材料和结构进行全面的优化设计，以实现最佳的辐射制冷性能。设计完成后，对所得辐射制冷产品的可行性和成本进行评估。

------

#### **参考文献：**

[1] LI X, ZHOU Y, YU S, et al. Urban heat island impacts on building energy consumption: a review of approaches and findings [J]. Energy, 2019, 174: 407-419. (城市热岛效应对建筑能耗的影响：方法与发现综述)

[2] SANTAMOURIS M, SYNNEFA A, KARLESSI T. Using advanced cool materials in the urban built environment to mitigate heat islands and improve thermal comfort conditions [J]. Solar Energy, 2011, 85(12): 3085-3102. (在城市建筑环境中使用先进的冷却材料以缓解热岛效应并改善热舒适条件)

[3] MANDAL J, YANG Y, YU N, et al. Paints as a scalable and effective radiative cooling technology for buildings [J]. Joule, 2020, 4(7): 1350-1356. (涂料作为一种可扩展且有效的建筑辐射制冷技术)

-----

#### 我的初步分析与总结

##### 主要目标

我们需要找到一个理想的材料，在可见光（$0.3 -2.5\mu m$）范围内吸收率$A$尽可能低，在大气窗口（$8-13\mu m$）范围内发射率$\varepsilon$尽可能高。

对于**问题1**（和**问题3**），首先要在网站上找到对应**PDMS**材料的复折射率$N(λ)=n(λ)+iκ(λ)$。其中 $n$ 是折射率，$κ$ 是消光系数。（这个好找，可以汇总）

然后基于这个数据来构建光线透射/辐射率模型。

**模型：** **传输矩阵法（Transfer Matrix Method, TMM）**

- 因为是“薄膜”，光在膜的上表面和下表面之间会发生干涉。

- 利用菲涅尔方程（Fresnel equations）构建矩阵，计算整个系统的**反射率** R(λ) 和**透射率** T(λ)。

- 根据基尔霍夫定律（Kirchhoff's Law），对于不透明基底或特定条件下，**吸收率等于发射率**：

  所以得到如下式子：

  $$ε(λ,d)=A(λ,d)=1−R(λ,d)−T(λ,d)$$

  > - $R$：反射率；
  >
  > - $T$：透射率。

  另外，有此式

  $\delta = \frac{2\pi N_{film} d \cos\theta}{\lambda}$

  以此可以得出实际上的相位厚度$\delta$。

  **结果：** 得到一个函数或曲线，显示随着 PDMS 厚度 d 的增加，发射率 ε 如何变化（通常厚度增加会提高发射率，但也会影响太阳光波段的吸收，需要权衡）。

> 公式存档：
>
> $$\begin{aligned}
> N_f(\lambda) &= n(\lambda) + i k(\lambda), \quad \delta(\lambda, d) = \frac{2\pi}{\lambda} N_f(\lambda)\, d \\
> r_{01} &= \frac{N_\text{air} - N_f}{N_\text{air} + N_f}, \quad
> r_{12} = \frac{N_f - N_\text{sub}}{N_f + N_\text{sub}} \\
> t_{01} &= \frac{2 N_\text{air}}{N_\text{air} + N_f}, \quad
> t_{12} = \frac{2 N_f}{N_f + N_\text{sub}} \\
> r_\text{tot}(\lambda, d) &= \frac{r_{01} + r_{12} e^{2 i \delta}}{1 + r_{01} r_{12} e^{2 i \delta}}, \quad
> t_\text{tot}(\lambda, d) = \frac{t_{01} t_{12}\, e^{i \delta}}{1 + r_{01} r_{12} e^{2 i \delta}} \\
> R(\lambda, d) &= \left| r_\text{tot}(\lambda, d) \right|^2, \quad
> T(\lambda, d) = \left| t_\text{tot}(\lambda, d) \right|^2 \frac{\Re(N_\text{sub})}{\Re(N_\text{air})} \\
> \varepsilon(\lambda, d) &= 1 - R(\lambda, d) - T(\lambda, d)
> \end{aligned}$$

> $TMM$法是用**矩阵相乘**模拟计算光波通过多层材料后的**反射率**和透射率。这为后面问题中的优化提供了结构基础，在此方法下可以使用退火、遗传、粒子群算法等等方法来优化。

对于**问题2**，总体式子可以这么总结：

$$P_{cool}=P_{IR\_rad}(T)-P_{atmosphere\_absorb}(T_{amp})-P_{sun}-\varepsilon$$

- $P_{cool}$：核心目标函数，因为最后我们要达到一个动态平衡，所以最终的要求是让它等于0；
- $P_{atmosphere\_absorb}(T_{amp})$：大气逆辐射加热物体的功率，这是因为有一部分物体向外辐射的热量被大气吸收了，$T_{amp}$是环境温度（大气温度）；
- $P_{IR\_rad}(T)$：物体向外发射的功率，这里的$T$代表着物体表面的温度；
- $P_{sun}$：太阳辐射加热物体的功率；
- $\varepsilon$：这个是干扰项，代表非辐射的热交换（对流/传导）

总体上就是**最大化向外发射功率**，最小化**太阳辐射的吸收率**，而关于大气逆辐射这一项，似乎是一个依赖量。

<img src="/home/ziyu/.config/Typora/typora-user-images/image-20251120114637190.png" alt="image-20251120114637190" style="zoom:50%;" />

> 这里我还没有细看，讨论时细说。

对于**问题3/4**的优化问题，需要指定一个目标函数。

通过组合不同材料和厚度，逼近右图中的**红色理想曲线**（方波）。

- **模型扩展：** 将步骤一的 TMM 模型扩展到多层。

- **优化算法：** 使用遗传算法（GA）、粒子群算法（PSO）或模拟退火算法。

  - **变量：** 每层的材料类型（从给定的库中选）、每层的厚度 $d_i$。

  - **目标函数（Cost Function）：** 最大化 $P_{cool}$。或者定义一个“品质因数”（Figure of Merit），比如：

    <img src="/home/ziyu/.config/Typora/typora-user-images/image-20251120115001817.png" alt="image-20251120115001817" style="zoom:33%;" />