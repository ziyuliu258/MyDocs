# Graph Attention Network

## 📐 **GAT的数学定义**

### **符号定义**

- **图**：$\mathcal{G} = (\mathcal{V}, \mathcal{E})$
  - $\mathcal{V}$：节点集合，在你的场景里是 **通道** $\{1, 2, ..., C\}$
  - $\mathcal{E}$：边集合，表示通道间的连接关系
  
- **节点特征**：
  - $\mathbf{h}_i \in \mathbb{R}^{d}$：第 $i$ 个节点（通道）的特征向量
  - $\mathbf{H} = [\mathbf{h}_1, \mathbf{h}_2, ..., \mathbf{h}_C]^T \in \mathbb{R}^{C \times d}$：所有节点的特征矩阵

- **邻接矩阵**：
  - $\mathbf{A} \in \mathbb{R}^{C \times C}$：$A_{ij} = 1$ 表示节点 $i$ 和 $j$ 相连

---

## 🎯 **GAT的核心：单层图注意力**

### **目标**

给定输入特征 $\mathbf{H}^{(l)} \in \mathbb{R}^{C \times d^{(l)}}$，输出新的特征 $\mathbf{H}^{(l+1)} \in \mathbb{R}^{C \times d^{(l+1)}}$

---

### **步骤1：线性变换**

首先对每个节点做线性投影：

$$\mathbf{h}'_i = \mathbf{W} \mathbf{h}_i$$

其中：
- $\mathbf{W} \in \mathbb{R}^{d' \times d}$：可学习的权重矩阵
- $\mathbf{h}'_i \in \mathbb{R}^{d'}$：投影后的特征

---

### **步骤2：计算注意力系数（核心！）**

对于节点 $i$ 和它的邻居 $j$，计算**原始注意力分数**：

$$e_{ij} = \text{LeakyReLU}\left(\mathbf{a}^T [\mathbf{h}'_i \| \mathbf{h}'_j]\right)$$

**详细解释**：

1. **拼接特征**：$[\mathbf{h}'_i \| \mathbf{h}'_j] \in \mathbb{R}^{2d'}$
   - $\|$ 表示拼接（concatenation）
   - 把节点 $i$ 和节点 $j$ 的特征向量首尾相连

2. **注意力机制**：$\mathbf{a} \in \mathbb{R}^{2d'}$ 是可学习的注意力向量
   - $\mathbf{a}^T [\mathbf{h}'_i \| \mathbf{h}'_j]$ 计算一个标量分数
   
3. **激活函数**：LeakyReLU 引入非线性

**物理意义**：

- $e_{ij}$ 衡量节点 $j$ 对节点 $i$ 的重要性
- 数值越大 → 节点 $j$ 对预测节点 $i$ 越重要

---

### **步骤3：归一化注意力权重**

用 **Softmax** 归一化，确保权重和为1：

$$\alpha_{ij} = \frac{\exp(e_{ij})}{\sum_{k \in \mathcal{N}_i} \exp(e_{ik})}$$

其中：
- $\mathcal{N}_i$：节点 $i$ 的邻居集合
- $\alpha_{ij} \in [0, 1]$：归一化后的注意力权重
- $\sum_{j \in \mathcal{N}_i} \alpha_{ij} = 1$

**举例**：

假设通道0有3个邻居：

| 邻居  | 原始分数 $e_{0j}$ | 归一化权重 $\alpha_{0j}$                         |
| ----- | ----------------- | ------------------------------------------------ |
| 通道1 | 2.5               | $\frac{e^{2.5}}{e^{2.5}+e^{1.8}+e^{0.5}} = 0.65$ |
| 通道2 | 1.8               | $\frac{e^{1.8}}{e^{2.5}+e^{1.8}+e^{0.5}} = 0.30$ |
| 通道3 | 0.5               | $\frac{e^{0.5}}{e^{2.5}+e^{1.8}+e^{0.5}} = 0.05$ |

---

### **步骤4：聚合邻居特征**

用注意力权重对邻居特征做**加权求和**：

$$\mathbf{h}^{(l+1)}_i = \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha_{ij} \mathbf{h}'_j\right)$$

其中：
- $\sigma$：激活函数（通常是 ReLU 或 ELU）
- $\mathbf{h}^{(l+1)}_i$：节点 $i$ 在下一层的特征

**直观理解**：
```
节点0的新特征 = 0.65 × 节点1的特征 + 0.30 × 节点2的特征 + 0.05 × 节点3的特征
```

---

## 🔥 **多头注意力（Multi-Head Attention）**

为了增强表达能力，GAT 使用 **$K$ 个独立的注意力头**：

$$\mathbf{h}^{(l+1)}_i = \Big\| _{k=1}^K \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha^k_{ij} \mathbf{W}^k \mathbf{h}_j\right)$$

或者在最后一层用**平均**代替拼接：

$$\mathbf{h}^{(l+1)}_i = \sigma\left(\frac{1}{K}\sum_{k=1}^K \sum_{j \in \mathcal{N}_i} \alpha^k_{ij} \mathbf{W}^k \mathbf{h}_j\right)$$

其中：
- $\|$ 表示拼接
- $K$ 是注意力头的数量（通常取 4、8 等）
- 每个头有独立的参数 $\mathbf{W}^k, \mathbf{a}^k$

---

## 📊 **完整的GAT层公式总结**

**输入**：$\mathbf{H} \in \mathbb{R}^{C \times d}$

**输出**：$\mathbf{H}' \in \mathbb{R}^{C \times d'}$

**步骤**：

1. **特征投影**：
   $$\mathbf{h}'_i = \mathbf{W} \mathbf{h}_i, \quad \forall i \in \{1, ..., C\}$$

2. **注意力分数**：
   $$e_{ij} = \text{LeakyReLU}(\mathbf{a}^T [\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j])$$

3. **归一化**：
   $$\alpha_{ij} = \frac{\exp(e_{ij})}{\sum_{k \in \mathcal{N}_i} \exp(e_{ik})}$$

4. **聚合**：
   $$\mathbf{h}^{\text{out}}_i = \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha_{ij} \mathbf{W} \mathbf{h}_j\right)$$

5. **多头拼接**（如果用多头）：
   $$\mathbf{H}' = \Big[\mathbf{H}'^{(1)} \| \mathbf{H}'^{(2)} \| ... \| \mathbf{H}'^{(K)}\Big]$$

---

## 🎓 **与Transformer注意力的对比**

### **Transformer的Self-Attention**

$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{Q}\mathbf{K}^T}{\sqrt{d_k}}\right)\mathbf{V}$$

### **GAT的图注意力**

$$\alpha_{ij} = \frac{\exp(\mathbf{a}^T[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j])}{\sum_{k \in \mathcal{N}_i} \exp(\mathbf{a}^T[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_k])}$$

**关键区别**：

| 特性       | Transformer                   | GAT                                                     |
| ---------- | ----------------------------- | ------------------------------------------------------- |
| 注意力范围 | **全局**（所有位置）          | **局部**（邻居节点）                                    |
| 注意力计算 | 点积 $\mathbf{Q}\mathbf{K}^T$ | 拼接 + MLP $\mathbf{a}^T[\mathbf{h}_i \| \mathbf{h}_j]$ |
| 适用场景   | 序列数据（文本、时间序列）    | 图数据（社交网络、分子）                                |

---

## 🧮 **用你的数据举例**

假设：
- $C = 7$ 个通道（温度、湿度、气压...）
- $d = 16$（embedding维度，你的 `d_model`）
- $K = 4$ 个注意力头

### **输入**
$\mathbf{H} \in \mathbb{R}^{7 \times 16}$，每行是一个通道的时间平均特征

### **参数**
- $\mathbf{W} \in \mathbb{R}^{4 \times 16}$（每个头投影到4维）
- $\mathbf{a} \in \mathbb{R}^{8}$（拼接后是8维，因为 $4 + 4 = 8$）

### **计算过程**

**对于通道0**：

1. **投影**：$\mathbf{h}'_0 = \mathbf{W} \mathbf{h}_0 \in \mathbb{R}^{4}$

2. **计算与通道1的注意力**：
   $$e_{01} = \text{LeakyReLU}(\mathbf{a}^T [\mathbf{h}'_0 \| \mathbf{h}'_1])$$
   $$= \text{LeakyReLU}([a_1, ..., a_8]^T \cdot [h'_{0,1}, ..., h'_{0,4}, h'_{1,1}, ..., h'_{1,4}])$$
   $$= \text{LeakyReLU}(2.34)  \quad \text{（假设算出来）}$$

3. **对所有6个邻居重复**：
   $$e_{00} = 3.1, \quad e_{01} = 2.34, \quad e_{02} = 1.8, \quad ...$$

4. **归一化**：
   $$\alpha_{01} = \frac{e^{2.34}}{e^{3.1} + e^{2.34} + e^{1.8} + ... } = 0.25$$

5. **聚合**：
   $$\mathbf{h}^{\text{out}}_0 = \text{ReLU}(0.5 \times \mathbf{h}'_0 + 0.25 \times \mathbf{h}'_1 + 0.15 \times \mathbf{h}'_2 + ...)$$

6. **4个头重复，然后拼接**：
   $$\mathbf{h}^{\text{final}}_0 = [\mathbf{h}^{\text{out},(1)}_0 \| \mathbf{h}^{\text{out},(2)}_0 \| \mathbf{h}^{\text{out},(3)}_0 \| \mathbf{h}^{\text{out},(4)}_0] \in \mathbb{R}^{16}$$

---

## 💡 **总结**

GAT的核心公式：

$$\boxed{\mathbf{h}^{(l+1)}_i = \sigma\left(\sum_{j \in \mathcal{N}_i} \underbrace{\frac{\exp(\mathbf{a}^T[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j])}{\sum_k \exp(\mathbf{a}^T[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_k])}}_{\text{注意力权重 } \alpha_{ij}} \mathbf{W}\mathbf{h}_j\right)}$$

**关键思想**：

1. 用神经网络学习节点间的重要性（注意力权重）
2. 用注意力权重聚合邻居信息
3. 端到端可微，可以反向传播

现在清楚了吗？需要我继续解释哪个部分？



非常好的问题！这是理解GAT的关键点。让我详细对比GAT的注意力机制和传统的Transformer注意力。

---

## 🔍 **GAT中的 $\mathbf{a}^T$ 是什么？**

### **完整公式回顾**

$$e_{ij} = \text{LeakyReLU}\left(\mathbf{a}^T [\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j]\right)$$

**符号解释**：

- $\mathbf{a} \in \mathbb{R}^{2d'}$：**可学习的注意力向量**（参数）
- $\mathbf{a}^T$：$\mathbf{a}$ 的转置，变成行向量 $[a_1, a_2, ..., a_{2d'}]$
- $[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j] \in \mathbb{R}^{2d'}$：拼接后的特征向量

---

## 📐 **$\mathbf{a}^T$ 的具体形式**

### **例子：假设 $d' = 4$**

经过 $\mathbf{W}$ 投影后：
- $\mathbf{W}\mathbf{h}_i = [h'_{i,1}, h'_{i,2}, h'_{i,3}, h'_{i,4}]^T \in \mathbb{R}^4$
- $\mathbf{W}\mathbf{h}_j = [h'_{j,1}, h'_{j,2}, h'_{j,3}, h'_{j,4}]^T \in \mathbb{R}^4$

**拼接**：
$$[\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j] = \begin{bmatrix}
h'_{i,1} \\
h'_{i,2} \\
h'_{i,3} \\
h'_{i,4} \\
h'_{j,1} \\
h'_{j,2} \\
h'_{j,3} \\
h'_{j,4}
\end{bmatrix} \in \mathbb{R}^8$$

**注意力向量**：
$$\mathbf{a} = \begin{bmatrix}
a_1 \\
a_2 \\
a_3 \\
a_4 \\
a_5 \\
a_6 \\
a_7 \\
a_8
\end{bmatrix} \in \mathbb{R}^8 \quad \text{（可学习参数）}$$

**计算注意力分数**：
$$e_{ij} = \text{LeakyReLU}\left(\mathbf{a}^T [\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j]\right)$$

$$= \text{LeakyReLU}\left(\sum_{k=1}^{8} a_k \cdot [\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j]_k\right)$$

$$= \text{LeakyReLU}\left(a_1 h'_{i,1} + a_2 h'_{i,2} + ... + a_8 h'_{j,4}\right)$$

**结果**：一个标量 $e_{ij} \in \mathbb{R}$

---

## ⚔️ **GAT vs Transformer Attention 对比**

### **1. Transformer的Self-Attention**

#### **公式**
$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{Q}\mathbf{K}^T}{\sqrt{d_k}}\right)\mathbf{V}$$

#### **详细步骤**

**输入**：$\mathbf{X} \in \mathbb{R}^{N \times d}$（N个token，每个d维）

**1. 计算 Q, K, V**：
$$\mathbf{Q} = \mathbf{X}\mathbf{W}_Q, \quad \mathbf{K} = \mathbf{X}\mathbf{W}_K, \quad \mathbf{V} = \mathbf{X}\mathbf{W}_V$$

其中：
- $\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V \in \mathbb{R}^{d \times d_k}$：可学习参数

**2. 计算注意力分数**（点积）：
$$\text{score}_{ij} = \frac{\mathbf{q}_i^T \mathbf{k}_j}{\sqrt{d_k}} = \frac{\sum_{m=1}^{d_k} q_{i,m} \cdot k_{j,m}}{\sqrt{d_k}}$$

**3. Softmax归一化**：
$$\alpha_{ij} = \frac{\exp(\text{score}_{ij})}{\sum_{k=1}^{N} \exp(\text{score}_{ik})}$$

**4. 加权求和**：
$$\mathbf{output}_i = \sum_{j=1}^{N} \alpha_{ij} \mathbf{v}_j$$

---

### **2. GAT的Attention**

#### **公式**
$$e_{ij} = \text{LeakyReLU}\left(\mathbf{a}^T [\mathbf{W}\mathbf{h}_i \| \mathbf{W}\mathbf{h}_j]\right)$$

#### **详细步骤**

**输入**：$\mathbf{H} \in \mathbb{R}^{N \times d}$（N个节点，每个d维）

**1. 线性变换**（只有一个W，相当于共享的Q和K）：
$$\mathbf{h}'_i = \mathbf{W}\mathbf{h}_i, \quad \mathbf{W} \in \mathbb{R}^{d' \times d}$$

**2. 计算注意力分数**（拼接 + MLP）：
$$e_{ij} = \text{LeakyReLU}\left(\mathbf{a}^T [\mathbf{h}'_i \| \mathbf{h}'_j]\right)$$

其中：
- $\mathbf{a} \in \mathbb{R}^{2d'}$：单个可学习向量（相当于一个单层MLP的权重）

**3. Softmax归一化**：
$$\alpha_{ij} = \frac{\exp(e_{ij})}{\sum_{k \in \mathcal{N}_i} \exp(e_{ik})}$$

注意：只对**邻居**归一化，不是全局！

**4. 加权求和**：
$$\mathbf{output}_i = \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha_{ij} \mathbf{h}'_j\right)$$

---

## 📊 **关键区别总结表**

| 特性           | **Transformer**                                       | **GAT**                                                      |
| -------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| **参数**       | $\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V$（3个矩阵） | $\mathbf{W}, \mathbf{a}$（1个矩阵 + 1个向量）                |
| **注意力计算** | **点积**：$\mathbf{q}_i^T \mathbf{k}_j$               | **拼接 + MLP**：$\mathbf{a}^T[\mathbf{h}'_i \| \mathbf{h}'_j]$ |
| **注意力范围** | **全局**（所有token）                                 | **局部**（只看邻居）                                         |
| **复杂度**     | $O(N^2 d_k)$                                          | $O(\|\mathcal{E}\| d')$（$\|\mathcal{E}\|$是边数）           |
| **归一化**     | 对所有位置归一化                                      | 只对邻居归一化                                               |
| **输出**       | 加权 $\mathbf{V}$                                     | 加权 $\mathbf{W}\mathbf{h}$                                  |

---

## 🤔 **为什么GAT不用QKV？**

### **原因1：图的稀疏性**

- **Transformer**：每个token和所有token都有关系（全连接）
  - 需要 Q（查询）和 K（键）来计算相似度
  
- **GAT**：每个节点只和邻居有关系（稀疏连接）
  - 已经知道哪些是邻居（邻接矩阵），不需要"检索"
  - 只需要学习"邻居有多重要"

### **原因2：计算效率**

**Transformer的点积**：
$$\text{score}_{ij} = \mathbf{q}_i^T \mathbf{k}_j = \sum_{m=1}^{d_k} q_{i,m} k_{j,m}$$

- 需要两个独立的投影（$\mathbf{W}_Q, \mathbf{W}_K$）
- 对称性好（$\mathbf{q}_i^T \mathbf{k}_j = \mathbf{k}_j^T \mathbf{q}_i$）

**GAT的拼接**：
$$e_{ij} = \mathbf{a}^T [\mathbf{h}'_i \| \mathbf{h}'_j]$$

- 只需要一个投影（$\mathbf{W}$）
- 可以捕捉**不对称关系**（$e_{ij} \neq e_{ji}$）
- 参数更少

---

## 🎯 **直观对比例子**

### **Transformer的方式（点积）**

假设 $d_k = 2$：

$$\mathbf{q}_i = \begin{bmatrix} 1.0 \\ 0.5 \end{bmatrix}, \quad \mathbf{k}_j = \begin{bmatrix} 0.8 \\ 0.6 \end{bmatrix}$$

$$\text{score}_{ij} = 1.0 \times 0.8 + 0.5 \times 0.6 = 1.1$$

**含义**：查询 $i$ 和键 $j$ 的"匹配度"

---

### **GAT的方式（拼接 + MLP）**

假设 $d' = 2$：

$$\mathbf{h}'_i = \begin{bmatrix} 1.0 \\ 0.5 \end{bmatrix}, \quad \mathbf{h}'_j = \begin{bmatrix} 0.8 \\ 0.6 \end{bmatrix}$$

$$[\mathbf{h}'_i \| \mathbf{h}'_j] = \begin{bmatrix} 1.0 \\ 0.5 \\ 0.8 \\ 0.6 \end{bmatrix}$$

$$\mathbf{a} = \begin{bmatrix} 0.2 \\ -0.3 \\ 0.5 \\ 0.1 \end{bmatrix}$$

$$e_{ij} = \text{LeakyReLU}(0.2 \times 1.0 - 0.3 \times 0.5 + 0.5 \times 0.8 + 0.1 \times 0.6)$$
$$= \text{LeakyReLU}(0.2 - 0.15 + 0.4 + 0.06) = \text{LeakyReLU}(0.51) = 0.51$$

**含义**：节点 $i$ 和 $j$ 组合后的"重要性"

---

## 🔧 **GAT可以改成QKV形式吗？**

**可以！** 这就是 **GATv2** 的改进：

### **原始GAT**：
$$e_{ij} = \mathbf{a}^T \text{LeakyReLU}(\mathbf{W}[\mathbf{h}_i \| \mathbf{h}_j])$$

### **GATv2**（更像Transformer）：
$$e_{ij} = \mathbf{a}^T \text{LeakyReLU}(\mathbf{W}_1 \mathbf{h}_i + \mathbf{W}_2 \mathbf{h}_j)$$

这里：
- $\mathbf{W}_1$ 相当于 $\mathbf{W}_Q$（查询）
- $\mathbf{W}_2$ 相当于 $\mathbf{W}_K$（键）
- $\mathbf{a}$ 相当于最后的投影

**优势**：
- 表达能力更强（动态注意力）
- 更接近Transformer的设计

---

## 💡 **总结**

### **GAT的 $\mathbf{a}^T$**

- 是一个**可学习的向量**（参数），维度是 $2d'$
- 作用：把拼接后的特征映射成**一个标量分数**
- 相当于一个**单层MLP的权重**

### **QKV的对比**

| 方面       | Transformer    | GAT                            |
| ---------- | -------------- | ------------------------------ |
| Q（查询）  | $\mathbf{W}_Q$ | 包含在 $\mathbf{a}$ 的前半部分 |
| K（键）    | $\mathbf{W}_K$ | 包含在 $\mathbf{a}$ 的后半部分 |
| V（值）    | $\mathbf{W}_V$ | $\mathbf{W}$（共享）           |
| 注意力函数 | 点积           | 拼接 + 线性                    |

### **为什么不同？**

- **Transformer**：全局注意力，需要高效检索（点积快）
- **GAT**：局部注意力，已知邻居，更关注关系建模（拼接灵活）

---

现在理解了吗？GAT的 $\mathbf{a}^T$ 就是一个简化的、单层的注意力机制，不需要显式的QKV分离！

