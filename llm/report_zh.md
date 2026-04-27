---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #FFFFFF
color: #303133
footer: Chen Qi @ 2026
style: |
  p {
    margin: 16px;
  }

  section {
    position: relative;
    font-family: Bahnschrift;
    padding-bottom: 64px !important;
    padding-top: 64px !important;
  }

  section.title-page {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  section::before {
    content: "";
    position: absolute;
    background-color: #F2F3F5;
    height: 64px;
    width: 100%;
    top: 0;
    left: 0;
  }

  section::after {
    content: "";
    background-color: #F2F3F5;
    margin-top: auto;
    height: 64px;
    width: 100%;

    display: flex;
    flex-direction: row-reverse;
    z-index: 0;
  }

  header {
    z-index: 1;
    color: #303133;
  }

  footer {
    z-index: 1;
    color: #303133;
  }

  img {
    max-height: 50vh;
    max-width: 100%;
  }

  strong {
    color: #409EFF !important;
  }

  code {
    background-color: #303133;
  }

  pre {
    background-color: #303133;
  }
---

<!-- _class: title-page -->

## 大语言模型（LLM）概述：
## **推理**与**优化**

---

<!-- _header: 目录 -->

- #### 变换器（Transformer）
  - 自然语言处理（NLP）要解决的问题
  - Transformer出现之前的方案
  - Transformer原理
- #### 大语言模型（LLM）
  - 面临的挑战
  - KV缓存（KV Cache）
- #### 优化策略
---

<!--
_class: title-page
_header: Transformer
-->

## 自然语言处理（NLP）：
## 我们**要解决什么**
## 以及**如何解决**

---

<!-- header: 自然语言处理（NLP） -->

- ###### 自然语言处理（NLP）要解决什么问题？

---

- ###### 自然语言处理（NLP）要解决什么问题？

自然语言处理最核心的应用场景是**机器翻译**。

在翻译任务中，我们需要将一句话转换成另一种语言的句子。

---

- ###### 自然语言处理（NLP）要解决什么问题？

自然语言处理最核心的应用场景是**机器翻译**。

在**机器翻译**中，我们需要将一句话转换成另一种语言的句子。

待翻译的句子和翻译结果可以是**任意内容**，例如：

![](assets/translate-example.png)

---

- ###### 自然语言处理（NLP）要解决什么问题？

![](assets/translate-example.png)

更形象地说，给定一个句子作为上下文，我们需要**预测接下来的内容**。

---

- ###### 如何将文字表示为计算机能**理解**的形式？

---

- ###### 如何将文字表示为计算机能**理解**的形式？
  
- **解决方案**：将文字表示为**向量**（嵌入向量，Embeddings）

![](assets/embedding.png)

- 借助预训练的矩阵，每个标记（token）会被转换为维度为$d_{model}$的向量，向量中的每个数值代表该标记的一个语义特征。

---

<!-- header: Transformer出现之前 -->

- ###### 那么在Transformer出现之前，我们如何处理嵌入向量的翻译？

---

- ###### 那么在Transformer出现之前，我们如何处理嵌入向量的翻译？

**循环神经网络（RNN, Recurrent Neural Network）** 是当时机器翻译的常用方案

![](assets/fully-connected-neuro-network.png)

**神经网络**可以将一个向量转换成另一个向量。

---

- ###### 循环神经网络（RNN）的工作原理？

在**全连接神经网络**的基础上，RNN能够记住之前所有嵌入向量在隐藏层的加权值。

这使得RNN可以逐个接收嵌入向量，并记住上下文信息。

![](assets/rnn-translate.png)

---

- ###### 循环神经网络（RNN）的工作原理？

![](assets/rnn-translate.png)

在接收完所有嵌入向量后，RNN能够记住所有标记的综合上下文信息。

随后，RNN可以输出一个向量，该向量代表每个标记作为下一个标记的概率。

同理，RNN接收一个标记后，会预测下一个标记。

---

<!-- header: 循环神经网络（RNN）的缺陷 -->

- 那么为什么RNN需要被替代？

RNN只能逐个接收嵌入向量，这意味着**必须处理完前一个标记，才能处理下一个标记**。

![](assets/rnn-translate.png)

这会导致处理速度极慢，且难以优化。

此外，随着RNN处理的标记数量增加，上下文信息可能会丢失。

---

<!--
_class: title-page
header: Transformer
-->

## Transformer：
## 一种**全新的方法**

###### 核心论文：Attention is All You Need（神经信息处理系统大会 2017）

---

- ###### Transformer如何改进RNN？

**RNN**按顺序处理数据（时间步$t$依赖于$t-1$）。
**Transformer**则**并行**处理数据。

1.  **并行性**：一次性接收整个句子，无需等待前一个词处理完成。
2.  **长程依赖**：通过**自注意力（Self-Attention）** 机制，第一个词可以直接“看到”最后一个词，不受距离限制。

---

<!-- _header: Transformer - 整体结构 -->

- ###### 整体框架：编码器-解码器（Encoder-Decoder）结构

Transformer遵循标准的**编码器-解码器**结构。

![](assets/transformer-overview.png)

---

<!-- _header: Transformer - 编码器（Encoder） -->

- ###### 深入解析编码器（Encoder）

编码器由$N$个相同的层堆叠而成（例如$N=6$）。
每一层包含两个核心子层：

1.  **多头自注意力（Multi-Head Self-Attention）**：捕捉标记之间的关联关系。
2.  **前馈神经网络（FFN, Feed-Forward Network）**：处理并消化捕捉到的信息。

![](assets/transformer-encoder-layer.png)

---

<!-- header: Transformer - 多头自注意力（Multi-Head Self-Attention） -->

![](assets/embedding.png)
![](assets/qkv.png)

---

- ###### 查询（Q）、键（K）、值（V）的计算逻辑
  - **查询（Query, $Q$）**：我想要找什么。
  - **键（Key, $K$）**：文件的标签/标识。
  - **值（Value, $V$）**：文件的实际内容。

- **注意力分数计算公式**：
$$Attention(Q, K, V) = Softmax(\frac{Q\times K^T}{\sqrt{d_k}})\times V$$

---

- ###### 注意力（Attention）的计算逻辑
  - **$Q\times K^T$**：找到与当前查询相关的标记。
  - **$Softmax$ & $\sqrt{d_k}$**：对分数进行归一化处理。
  - **$\times V$**：提取匹配标记的语义信息。

- **注意力分数计算公式**：
$$Attention(Q, K, V) = Softmax(\frac{Q\times K^T}{\sqrt{d_k}})\times V$$

---

- ###### 为什么需要“多头（Multi-Head）”？

自然语言具有复杂性。

---

- ###### 为什么需要“多头（Multi-Head）”？

自然语言具有复杂性。

我们将$Q、K、V$拆分为`n_heads`个注意力头，独立计算每个头的关联关系，再将结果拼接。

![](assets/multi-heads.png)

---

<!-- header: Transformer - 前馈神经网络 & 残差连接 & 层归一化 -->

注意力机制负责**从其他标记收集信息**，而前馈神经网络（FFN）负责**独立处理信息**。

- **前馈神经网络（FFN）**： 
  - 一个简单的多层感知机（MLP），用于消化上下文信息。
  - 先扩展维度（例如$512 \to 2048$），再压缩回原维度。

- **残差连接（Add）& 层归一化（Norm）**： 
  - **残差连接（Add）**：$x + Layer(x)$，防止信息丢失。
  - **层归一化（Norm, LayerNorm）**：稳定训练过程中的数值分布。

---

<!-- header: Transformer - 解码器（Decoder） -->

- ###### 编码器（Encoder）负责理解已有内容（上下文），解码器（Decoder）负责预测后续内容（生成）。

- **核心目标**：自回归生成（Autoregressive Generation）。
给定标记$x_1, ..., x_{t-1}$，预测$x_t$。

![](assets/decoder-layer.png)

![](assets/decoder.png)

---

- ###### 深入解析解码器（Decoder）：生成过程

解码器的层结构与编码器类似：

![](assets/translate-loop.png)

---

<!-- header: Transformer: 解码器的多头自注意力 -->

假设我们已经生成了“I”“Love”，现在输入为“AI”。
我们需要仅针对这个**新标记**计算查询（Q）、键（K）和值（V）。

$$
q_{new} = x_{AI} \times W_Q, \quad k_{new} = x_{AI} \times W_K, \quad v_{new} = x_{AI} \times W_V
$$

- **$q_{new}$**：新的查询向量，“哪些内容与‘AI’相关？”
- **$k_{new}$**：新的键向量，“AI”的标识。
- **$v_{new}$**：新的内容向量，“AI”的语义。

---

为了预测下一个词，“AI”必须“回顾”“I”和“Love”。
我们将**历史键（Past Keys）** 与**新键（New Key）** 合并。

所有标记的**值（Values）** 保持不变。

$$
K_{total} = Concat([K_{past}, k_{new}])
$$

$$
V_{total} = Concat([V_{past}, v_{new}])
$$

---

计算**新查询（New Query）** 与**所有键（历史+当前）** 的匹配度。

$$
Scores = q_{new} \times K_{total}^T
$$

最终，计算解码器的**输出向量**。

$$
Output = Softmax(\frac{Scores}{\sqrt{d_k}}) \times V_{total}
$$

- 该向量包含了预测下一个词所需的**完整上下文信息**。

---

经过上述所有步骤后，输出向量会被映射到词汇表维度，得到各标记的概率。

$$
Probabilities = Softmax(Output \times W_{vocab})
$$

![](assets/probabilities.png) 

---

<!-- header: Transformer: KV缓存（KV Cache） -->

- ###### 朴素推理策略（无缓存）

我们先看标准Transformer在**无任何优化**时的文本生成过程。
要生成第$t$步的标记，必须输入**整个序列**$x_1, ..., x_{t-1}$。

1. **第一步**：输入`["I"]` → 计算1个标记的注意力。
2. **第二步**：输入`["I", "Love"]` → 计算2个标记的注意力。
3. **第三步**：输入`["I", "Love", "AI"]` → 计算3个标记的注意力。

---

**你发现问题了吗？**
第一步中我们计算了“I”的嵌入向量和注意力，
第二步又**重新计算**了一次，
第三步**再次重复计算**。

**随着序列长度增加，这种“冗余计算”会形成一个巨大的金字塔。**

---

我们来计算生成长度为$N$的序列的开销。

在**第$t$步**（当前序列长度为$t$）：

---

我们来计算生成长度为$N$的序列的开销。

在**第$t$步**（当前序列长度为$t$）：
* 输入矩阵的形状为$(t \times d)$。

---

我们来计算生成长度为$N$的序列的开销。

在**第$t$步**（当前序列长度为$t$）：
* 输入矩阵的形状为$(t \times d)$。
* **自注意力计算开销**：$Q(t \times d) \times K^T(d \times t) \to \text{分数矩阵}(t \times t)$。

---

我们来计算生成长度为$N$的序列的开销。

在**第$t$步**（当前序列长度为$t$）：
* 输入矩阵的形状为$(t \times d)$。
* **自注意力计算开销**：$Q(t \times d) \times K^T(d \times t) \to \text{分数矩阵}(t \times t)$。
* 矩阵乘法复杂度：**$O(t^2 \cdot d)$**。

---

我们来计算生成长度为$N$的序列的开销。

在**第$t$步**（当前序列长度为$t$）：
* 输入矩阵的形状为$(t \times d)$。
* **自注意力计算开销**：$Q(t \times d) \times K^T(d \times t) \to \text{分数矩阵}(t \times t)$。
* 矩阵乘法复杂度：**$O(t^2 \cdot d)$**。

**总推理开销**（所有步骤之和）：
$$
\text{总开销} = \sum_{t=1}^{N} O(t^2 \cdot d) \approx O(N^3 \cdot d)
$$

---

- ###### 核心总结：为什么需要优化？

标准Transformer面临的核心问题：

| 问题 | 表现 | 根源 | 解决方案 |
| :--- | :--- | :--- | :--- |
| **冗余计算** | 推理速度慢（复杂度$O(N^3)$） | 每一步都重新计算历史标记的K、V | **KV缓存（KV Cache）** |

---

<!--
header: 复杂度分析：逐步骤拆解
_class: title-page
-->

## 真实开销：
## 时间 & 空间 拆解

---

<!--
header: 复杂度分析：编码器层（Encoder Layer）
-->

- **QKV计算**：
  - **时间复杂度**：$[n, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(nd_{model}^2)$
- **自注意力计算**： 
  - **时间复杂度**：$[n, d_{heads}]\times [d_{heads}, n] \times [n, d_{heads}] \Rightarrow O(n^2d_{heads})$
- **前馈神经网络（FFN）**：
  - **时间复杂度**：$[n, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(nd_{model}^2)$

- **总计**：
  - **时间复杂度**：$O(nd^2 + n^2d)$

---

<!--
header: 复杂度分析：解码器层（Decoder Layer）
-->

- **QKV计算**：
  - **时间复杂度**：$[1, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(d_{model}^2)$
- **自注意力计算**： 
  - **时间复杂度**：$[1, d_{heads}]\times [d_{heads}, n+t] \times [n+t, d_{heads}] \Rightarrow O(td_{heads})$
- **前馈神经网络（FFN）**：
  - **时间复杂度**：$[1, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(d_{model}^2)$

- **总计（生成长度为m的序列）**：
  - **时间复杂度**：$O(m(d^2 + (n+m)d))$

---

<!--
header: Transformer: 训练过程
_class: title-page
-->

## Transformer：
## 如何训练

###### 从随机初始化到具备智能

---

- ###### 核心转变

**训练** = 调整矩阵参数（$W$）以最小化误差。

![](assets/training-overview.png)

---

- ###### 第一步：从原始文本构建“问题”（输入）和“答案”（目标）。

![](assets/train-target.png)

---

- ###### 第二步：“作弊”问题

如果输入整个序列，位置1的“I”能看到位置2的“Love”，
这属于**数据泄露（Data Leakage）**。

![](assets/cheating.png)

---

- ###### 第三步：掩码矩阵（遮眼罩）

我们给注意力分数矩阵的上三角部分加上$-\infty$，阻断对未来标记的关联。

$$
\text{注意力分数} + \text{掩码} \rightarrow \text{Softmax归一化}
$$

$$
\begin{bmatrix} 
\text{相似度}(I, I) & \text{相似度}(I, Love) \\ 
\text{相似度}(Love, I) & \text{相似度}(Love, Love) 
\end{bmatrix}
+
\begin{bmatrix} 
0 & \mathbf{-\infty} \\ 
0 & 0 
\end{bmatrix}
\approx
\begin{bmatrix} 
\text{分数} & \mathbf{0} \\ 
\text{分数} & \text{分数} 
\end{bmatrix}
$$

* **结果**：
  * 第一行只能**看到**第一列。
  * 第二行能看到第一列和第二列。

---

- ###### 第四步：并行损失计算

我们同时计算**所有**位置的误差。

![](assets/loss.png)

$$
\text{总损失} = \text{损失}_1 + \text{损失}_2 + \text{损失}_3
$$

---

- ###### 第五步：学习循环

![](assets/learning.png)

**该循环会重复数十亿次。**

---

- ###### 对比：训练 vs 推理

| 特征 | 训练 | 推理 |
| :--- | :--- | :--- |
| **策略** | **并行**（教师强制，Teacher Forcing） | **串行**（自回归） |
| **输入** | 完整句子 | 仅历史标记 |
| **未来标记** | **掩码屏蔽**（人工处理） | 不存在 |
| **复杂度** | $O(N^2)$（单次遍历） | $O(N^3)$（循环生成） |

---

<!--
header: Transformer: 内存IO
_class: title-page
-->

## Transformer：
## 内存IO

###### 硬件层面的实际开销

---

理解**数据移动**是优化的关键。

![](assets/memory.png)

**核心规则**：计算只能在静态随机存取存储器（SRAM）中进行。
数据必须从高带宽内存（HBM）加载到SRAM才能处理，处理后再写回HBM。

---

- 自注意力的IO复杂度：

| 操作               | 从HBM读取           | 写入HBM   | 复杂度            |
| ---------------- | -------------- | ------ | -------------- |
| $S = QK^T$        | Q、K（2Nd）     | S（N²） | O(Nd + N²)     |
| $P = softmax(S)$ | S（N²）         | P（N²） | O(N²)          |
| $O = PV$         | P、V（N² + Nd） | O（Nd） | O(Nd + N²)     |
| **总计**     |                  |        |         **O(Nd + N²)** |

数据在HBM和SRAM之间频繁移动的开销**极高**。

---

<!--
header: Transformer: FlashAttention优化
_class: title-page
-->

## Transformer优化：
## FlashAttention

###### 具备IO感知的快速、内存高效的精确注意力机制

---

- ###### 阶段1：预填充（QKV计算）

**目标**：计算$S = QK^T, P = Softmax(S), O = PV$

1. 从HBM读取$Q、K$ → 计算$S$ → **将$S$写入HBM**。
2. 从HBM读取$S$ → 计算$P$ → **将$P$写入HBM**。
3. 从HBM读取$P、V$ → 计算$O$ → 将$O$写入HBM。

数据在HBM和SRAM之间频繁移动的开销**极高**。

**性能瓶颈**：向HBM读写$N^2$规模的矩阵。

---

- ###### FlashAttention：核心思想

**“IO感知”**：最小化HBM的访问次数。

两大核心技术：
1. **分块（Tiling）**：按块计算注意力，将小块的Q、K、V加载到SRAM中计算，更新输出时无需将完整的S矩阵写入HBM。
2. **重计算（Recomputation）**：反向传播时不保存注意力矩阵，而是实时重新计算。

---

<!-- header: FlashAttention: 分块（Tiling） -->

- ###### 技术1：分块（前向传播）

不再计算完整的$N \times N$矩阵，而是将$Q、K、V$拆分为能**适配SRAM**的小块。

---

<!-- header: 分块：如何计算？ -->

- ###### Softmax的问题

标准Softmax需要**整行数据**来计算归一化因子（分母）。

$$Softmax(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{N} e^{x_j}}$$

---

<!-- header: Softmax：块合并 -->

![](assets/merge-block.png)

如果对矩阵分片，**就无法获取全局求和结果**！

---

- ###### 解决方案：在线Softmax（Online Softmax）

我们可以**增量式**更新Softmax结果。
跟踪局部统计量：**最大值（$m$）** 和**求和值（$l$）**。

![](assets/online-softmax.png)

**核心**：当发现新的最大值时，按比例缩小历史求和值，保证计算逻辑正确。

---

向量$x\in R^B$的Softmax计算方式：

- $m(x) = \underset{i}{\max}\ x_i$
- $f(x) = [e^{x_1 - m(x)}\ \ldots\ e^{x_B - m(x)}]$
- $l(x) = \underset{i}{\Sigma}f(x)_i$
- $softmax(x) = \frac{f(x)}{l(x)}$

---

对于两个向量**$x^{(1)},x^{(2)}\in \mathbf{R}^B$**，拼接后的向量$x = [x^{(1)}, x^{(2)}]\in \mathbf{R}^{2B}$的Softmax可分解为：

- $m(x) = \max(m(x^{(1)}), m(x^{(2)}))$
- $f(x) = [e^{m(x^{(1)}) - m(x)}f(x^{(1)})\ \ e^{m(x^{(2)}) - m(x)}f(x^{(2)})]$
- $l(x) = e^{m(x^{(1)}) - m(x)}l(x^{(1)}) + e^{m(x^{(2)}) - m(x)}l(x^{(2)})$

- $P = Softmax(x) = \frac{f(x)}{l(x)}$

---

<!-- header: Transformer: FlashAttention优化 -->

- ###### 计算部分输出

一旦得到当前块的注意力概率（$P_{ij}$），将其与值（$V_j$）相乘。

$$O_{ij} = P_{ij} \times V_j$$

但不能直接将该结果与历史结果“相加”，因为**Softmax的分母发生了变化**！

---

- ###### 重新缩放与合并

要将**历史输出**与**新的部分输出**合并，必须根据更新后的Softmax统计量**重新缩放**历史数据。

![](assets/output.png)

---

**更新公式**：

$$
O_{new} = \frac{1}{\ell_{new}} \left( \underbrace{\ell_{old} e^{m_{old} - m_{new}} O_{old}}_{\text{重新缩放的历史输出}} + \underbrace{e^{\tilde{m_{ij}} - m_{new}} \tilde{P_{ij}} V}_{\text{新块输出}} \right)
$$

- **$\tilde{m_{ij}}, \tilde{P_{ij}}$**：$S_{ij}$的行最大值（rowmax）和$e^{S_{ij} - \tilde{m_{ij}}}$。

> **含义**：根据最大值的增幅比例“压缩”历史结果，再加上新块的加权贡献。

---

```python
# 从HBM加载Q、K、V，并拆分为块
for j in range(Tc):  # 外层循环：加载K、V块（适配SRAM）
    K_j, V_j = load_from_HBM(j) 
    for i in range(Tr):  # 内层循环：加载Q块（适配SRAM）
        Q_i, O_i, l_i, m_i = load_from_HBM(i)
        # 1. 计算注意力分数
        S_ij = Q_i @ K_j.T
        # 2. 计算局部Softmax统计量
        m_tilde = rowmax(S_ij)  # 行最大值
        P_tilde = exp(S_ij - m_tilde)
        l_tilde = rowsum(P_tilde)  # 行求和值
        # 3. 更新全局统计量（在线Softmax）
        m_new = max(m_i, m_tilde)
        l_new = exp(m_i - m_new) * l_i + exp(m_tilde - m_new) * l_tilde
        # 4. 重新缩放并更新输出（对应上述公式）
        O_i = (1 / l_new) * ( 
            l_i * exp(m_i - m_new) * O_i + 
            exp(m_tilde - m_new) * (P_tilde @ V_j) 
        )
        # 5. 写回HBM
        write_to_HBM(O_i, l_new, m_new)
```

---

- ###### 分析：为什么分块能减少HBM访问？

我们对比**HBM访问次数**（数据移动量）。

**标准注意力机制**：

1.  读取$Q、K$（$N\times d$）→ 写入$S$（$N^2$）。
2.  读取$S$（$N^2$）→ 写入$P$（$N^2$）。
3.  读取$P、V$（$N^2$、$N\times d$）→ 写入$O$（$N\times d$）。

**HBM总访问量**：$O(Nd + N^2)$

---

- ###### 分析：FlashAttention的IO复杂度

1. 块大小：**$B_c = B_r = O(\frac{M}{d})$**
2. 将K、V块加载到SRAM（**$\frac{N}{B_c}\times 2B_cd = O(Nd)$**）。
3. 将Q块加载到SRAM（**$\frac{N}{B_c}\times\frac{N}{B_r}\times B_rd = O(\frac{N^2d^2}{M})$**）。
4. 计算并写入O块（访问量与加载Q块相同）。

**HBM总访问量**：$O(Nd + N^2 d^2 M^{-1})$，其中$M$为SRAM容量

- 由于$d^2 \ll M$，HBM访问次数会显著减少。

---

<!--
header: Transformer: 块稀疏FlashAttention优化
_class: title-page
-->

## 扩展方向：
## 块稀疏FlashAttention

###### 能否扩展到无限上下文长度？

---

  - ###### 如果跳过部分块会怎样？

对于极长序列（如64k+），即使$O(N^2)$的计算量也过于庞大。

**观察结论**：注意力矩阵通常是稀疏的（大部分值接近0）。

**核心思路**：使用预定义的掩码$\tilde{M}$，如果掩码中某块的值为0，则**跳过该块的加载和计算**。

---

- ###### 块稀疏算法

![](assets/block-sparse.png)

---

- ###### 块稀疏复杂度

设$s$为非零块的占比。

- **标准FlashAttention**：
  - HBM访问量：$O(N^2 d^2 M^{-1})$

- **块稀疏FlashAttention**：
  - HBM访问量：$O(N^2 d^2 M^{-1} s)$

* 对于长序列$N$，$s$通常为$N^{-1/2}$或$N^{-1}\log N$。

---

<!--
header: Transformer: PagedAttention优化（vLLM）
_class: title-page
-->

## 优化策略：
## PagedAttention（vLLM）

###### 打破连续内存的限制

---

- ###### 解码阶段的内存问题

现有系统中，KV缓存需要**连续的内存空间**。
但我们无法预知输出序列的长度！

**结果**：必须**预分配**最大长度的内存（如2048）。

* **内部碎片**：预留但未使用的内存槽位。
* **外部碎片**：内存分配器无法找到大段连续内存。
* **最终结果**：GPU内存的**60% - 80%** 被浪费！

---

  - ###### 解决方案：虚拟内存

灵感来源于**操作系统虚拟内存（分页机制）**。

![](assets/logical-memory.png)

---

  - ###### 基于块的注意力计算

我们修改注意力计算逻辑，通过块表（Block Table）逐块获取数据。

对于嵌入向量**$x_i$**，计算：
$$q_i = W_qx_i, k_i = W_kx_i, v_i = W_vx_i$$

随后计算注意力和输出：

$$
a_{ij} = \frac{\exp(q_i^T k_j / \sqrt{d})}{\sum_{t=1}^{i} \exp(q_i^T k_{t} / \sqrt{d})}, o_{ij} = \Sigma_{j=1}^{i} a_{ij} v_{j}
$$

$$(Softmax(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{N} e^{x_j}})$$

---

- **流程**：
  1. 确定当前请求对应的逻辑块。
  2. 在**块表（Block Table）** 中查找物理地址。
  3. 将非连续的块从HBM加载到SRAM。
  4. 计算注意力。

---

- ###### 内存共享

由于使用块表，我们可以将**不同的逻辑块**映射到**同一个物理块**。

**典型场景**：并行采样 / 束搜索（Beam Search）。

  * 提示词（Prompt）：*"Translate this article..."*（共享）
  * 采样结果A：*"The article..."*
  * 采样结果B：*"This text..."*

---

- ###### 实现机制：写时复制（CoW, Copy-on-Write）

![](assets/copy-on-write.png)

---

- ###### 空间复杂度：近乎零浪费

  * **预分配方式**：$O(\text{最大序列长度} \times \text{批次大小})$ $\rightarrow$ **极度浪费**。
  * **PagedAttention方式**：$O(\text{实际序列长度} \times \text{批次大小})$。

**碎片分析**：

  * 无外部碎片（所有块大小相同）。
  * 内部碎片仅存在于**最后一个块**。
      * 浪费率 < 块大小 / 序列长度。
      * 当块大小为16时，浪费率**< 4%**。

---

- ###### 时间复杂度 & 额外开销

块表查找会降低速度吗？

  * **内核开销**：由于内存间接寻址和额外分支逻辑，注意力内核会产生20-26%的小幅开销。
  * **端到端收益**：
      * 内存利用率提升后，可大幅增加**批次大小（Batch Size）**。
      * **吞吐量**：比标准系统（如FasterTransformer）高2-4倍。

---

<!--
header: Transformer: CacheBlend优化
_class: title-page
-->

## 优化策略：
## CacheBlend

###### 高效处理多轮对话 & 检索增强生成（RAG）上下文

---

- ###### 前缀缓存（Prefix Caching）的局限性

前缀缓存在复用文本位于**最开头**时效果很好，但在检索增强生成（RAG）场景中，我们会检索多个文本块。

**输入结构**：`[文本块A] + [文本块B] + [文本块C] + [查询语句]`

  * **前缀缓存**：仅能复用`[文本块A]`的KV缓存。
  * **文本块B和C**：由于位置变化，且会关联新的前置文本，其KV缓存无法复用。

---

![](assets/cache.png)

---

- ###### 为什么不直接拼接预计算的KV缓存？

如果分别预计算文本块A和文本块B的KV缓存，再将它们拼接（“全KV复用”）：

**会丢失交叉注意力（Cross-Attention）**！
文本块B在预计算时从未“看到”文本块A。

![](assets/attention-loss.png)

---

- ###### 核心思路：选择性KV重计算

我们无需重新计算**全部**标记（100%），
也不能完全不重新计算（0%，生成质量低）。

**解决方案**：仅重新计算受新上下文影响最大的**小部分标记（约15%）** 的KV缓存。

$$
KV_{new} \approx KV_{预计算} + \text{更新}(选定标记)
$$

---

- ###### 选择哪些标记？

**核心洞察**：注意力具有**稀疏性**。
文本块B中仅有少数标记与文本块A存在强注意力关联。

这些标记被称为**高KV偏差标记（HKVD, High KV Deviation）**。

![](assets/selective-update.png)

---

- ###### 识别HKVD标记

若不进行完整计算，无法获取真实的偏差值。
**近似方案**：使用轻量级代理指标或渐进式过滤。

1.  在第一层计算注意力。
2.  选择注意力更新值最高的Top-K标记。
3.  将这些标记的索引传递到更深层（HKVD标记在各层高度相关）。

> **结果**：仅更新最关键的标记，即可恢复生成质量。

---

- ###### 隐藏开销：IO vs 计算

我们确定需要重新计算约15%的标记，
这会增加额外延迟吗？

1.  从存储设备（SSD/CPU内存）**加载**预计算的KV缓存到GPU。
2.  **重新计算**选定的HKVD标记。

$$\text{总延迟} = \sum_{i=1}^{L} (T_{加载}^{(i)} + T_{计算}^{(i)})$$

**问题**：如果串行执行，每一步都会增加延迟。

---

- ###### 解决方案：流水线（Pipelining）

**核心洞察**：GPU计算和内存加载（IO）可并行执行。

当GPU忙于**重新计算**第$i$层时，可从存储设备获取第$i+1$层的KV缓存。

---

- ###### 加载控制器

如何确保重新计算的耗时不超过加载耗时？
我们引入**加载控制器**来平衡两者：

$$T_{重新计算}(r\%) \approx T_{加载}(\text{存储设备})$$

**两种自适应策略**：

1.  **调整重计算比例**：若IO速度慢，可提高重计算比例（不超过上限），因为此时有“空闲”时间。
2.  **选择存储设备**：若固定重计算比例为15%，选择刚好能掩盖计算耗时的**成本最低**的存储设备（如SSD）。

---

由于将计算开销隐藏在IO耗时中，我们可将KV缓存存储在**速度更慢、成本更低**的介质中，而非昂贵的GPU HBM或CPU内存。

![](assets/select-ssd.png)
