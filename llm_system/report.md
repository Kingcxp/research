---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #FFFFFF
color: #303133
footer: Papers for building LLM system | Chen Qi @ 2026
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
    max-height: 70vh;
    max-width: 100%;
  }

  .whole-page {
    max-height: 70vh !important;
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
  
  td {
    font-size: 24px;
  }
---

<!-- 
_class: title-page
_header: Papers for building LLM system
-->

## **Papers for building LLM system**

---

<!--
_class: title-page
header: Efficient Cooperation-Aware Key and Value Management for LLM Inference
-->

## **CoKV: Management for LLM Inference**

https://github.com/ZJU-DIVER/CoKV/blob/main/full_paper.pdf

---

### **问题痛点**

* 在长文本场景下，KV 缓存的显存占用甚至可能超过模型本身的显存占用
* 现有的优化方法存在致命问题：
  * 要么用**完全相同的规则**压缩 KV 缓存，忽略注意力头的**重要性差异**
  * 要么**独立评估**注意力头的重要性，忽略注意力头之间的**协同工作**

本文提出 CoKV，旨在用**合作博弈论**的方法，精准衡量每个注意力头在协同工作中的真实贡献，精准衡量每个注意力头在协同工作中的真实贡献

---

### **合作博弈论与 Shapley 值**

Shapley 值能公平地算出每个人在所有可能的组队组合里，带来的**边际贡献平均值**

问题在于，Shapley 值的计算是一个 `P-Hard` 问题

---

### **合作博弈论与 Shapley 值**

传统的 Shapley 值计算方案：

$$
SV_i = \frac{1}{n} \sum_{S \subseteq N \setminus \{p_i\}} \frac{U(S \cup \{p_i\}) - U(S)}{\binom{n-1}{|S|}}
$$

表示**算头i加入任意组合S后的边际贡献，但要算 $2^n$ 次**
$U(S \cup i) - U(S)$ 表示**算头i加入组合S后的边际贡献**

时间复杂度：$\mathbf{O(T\cdot n\cdot C)}$

$\mathbf{C}$：置信度常数 $\frac{\ln(2n/\delta)}{\epsilon^2}$，$T$：单次算$\mathbf{U(S)}$的时间

---

### **优化 1：互补贡献替代边际贡献**

$$
SV_i = \frac{1}{n} \sum_{S \subseteq N \setminus \{p_i\}} \frac{U(S \cup \{p_i\}) - U(N \setminus (S \cup \{p_i\}))}{\binom{n-1}{|S|}}
$$

使用**互补贡献**$U(S) - U(N \setminus S)$，可以一次性更新 $S$ 中所有头的贡献

时间复杂度：$\mathbf{O(T\cdot C)}$

---

### **优化 2：递归计算**

$$
SSV_i^L = \frac{1}{|L|} \sum_{j=1}^n SV_{i,j} \cdot I_L(j)
$$

同一个头，在不同联盟大小下的**互补贡献高度相关**

因此不用遍历所有联盟大小（j=1~n），只挑几个代表性大小（切片 $\mathbf{L}$） 就能近似。

时间复杂度中的所有 $\mathbf{N}$ 替换为 $\mathbf{||L||}$

---

### **优化 3：蒙特卡洛采样近似**

使用随机采样 $\mathbf{M}$ 次联盟代替遍历所有联盟。

时间复杂度不变，但计算量显著减少

### **误差边界**
$$
|SV_i - SSV_i^L| \leq (b-a) \sqrt{\frac{(n-|L|+1)\ln(2/\delta)}{2|L|n}}
$$
- 含义：切片数量$|L|$越多，误差越小，理论保证准确性。

---

### **Shapley 计算（论文算法 1）**

```pseduo
1 初始化：每个头的SSV=0，每个联盟的贡献SV=0，计数m=0
2 循环M次（随机采样）：
3    生成所有头的随机排序πₖ
4    从切片L里随机选一个联盟大小j
5    取前j个头组成联盟S，剩下的头为H\S
6    计算U(S)：只用S里的头，模型效果得分
7    计算U(H\S)：只用H\S里的头，模型效果得分
8    计算互补贡献u = U(S) - U(H\S)
9    对S里的每个头i：
10       SVᵢⱼ += u （累加贡献）
11       mᵢⱼ += 1 （累加计数）
12 最终计算：每个头的SSVᵢᴸ = 平均(SVᵢⱼ/mᵢⱼ) / |L|
```

---

### **KV 缓存驱逐**

按重要性分数分配缓存大小，只保留每个头里最重要的 KV 对

$$
c_i = B \cdot \frac{NSV_i^L}{\sum_{j=1}^n NSV_j^L} + s
$$
- $B$：全局共享缓存预算（总显存）
- $NSV_i^L$：头$i$的归一化重要性分数（0~1）
- $s$：本地窗口大小（论文默认8，每个头必留的最小缓存）
- $c_i$：头$i$最终的KV缓存大小

---

### 论文算法 2

```
1 计算本地窗口的查询Q^win_i
2 计算本地窗口对所有KV的注意力分数A_i
3 聚合注意力分数（池化+均值）
4 用算法1+公式6，算出每个头的缓存大小c_i
5 选注意力分数最高的c_i个KV对
6 拼接：选中的外部KV + 本地窗口KV
7 输出最终保留的KV缓存
```

---

### **扩展方案：KV 缓存量化**

用**不同比特数**存KV，重要头用高精度（多比特），不重要头用低精度（少比特）。
1. 按$SSV_i^L$排序头；
2. 重要头分配**高比特**（如8bit），不重要头分配**低比特**（如4bit）；
3. 按统一量化公式压缩KV，还原时反量化。

---

### **CoKV 整体工作总流程图**

![](./assets/CoKV.png)

---

<!--
_class: title-page
header: AlayaDB: The Data Foundation for Efficient and Effective Long-context LLM Inference
-->

## **AlayaDB**
### Efficient and Efective Long-context LLM Inference

---

* 大模型（LLM）处理**超长文本**时，有 3 个关键指标很难同时做好：

1. **显存占用**：太长的文本会占满显卡
2. **推理速度**：生成每个字都很慢
3. **回答质量**：压缩了太多信息，并且为了保证速度，导致回答质量下降

* 现有方案：

- **耦合架构**：质量好，但占用显存很多并且很慢
- **KV 缓存分离**：速度中等，但依旧占用显存
- **稀疏注意力**：省显存，但是质量下降严重

---

### DIPR（动态内积范围查询）

- 之前的稀疏注意力都使用 Top-K，只保留前 K 个重要的 token
- **不同层，不同头**所需要的 token 差异实际上很大
- **不同任务**所需要的 token 差异也很大

因此，DIPR：

- 不固定取多少个 token
- 只设定一个 **“重要性门槛”**
- 自动抓所有超过这个门槛的 token
- 不同层、不同任务自动抓不同数量

---

- $q_i \in \mathbb{R}^{1 \times d}$：当前要生成第i+1个token时的查询向量
- $k_j \in \mathbb{R}^{1 \times d}$：第j个历史token的键向量
- $v_j \in \mathbb{R}^{1 \times d}$：第j个历史token的值向量
- $d$：向量维度
- $a_{ij}$：第j个历史token对第i+1个token的注意力分数
- 注意力分数计算公式（论文式1）：
  $$
  z_{ij} = \frac{q_i \cdot k_j^T}{\sqrt{d}}; \quad a_{ij} = softmax(z_{ij}) = \frac{exp(z_{ij})}{\sum_{t=1}^i exp(z_{it})}
  $$

---

**定义 1：基于注意力分数的关键 token**

> 给定所有键向量 $K = [k_1, \cdots, k_n]$，键 $k_j$ 是查询 $q_i$ 的关键token，当且仅当：
> $$
> a_{ij} \geq \alpha \times \max_{s \in [1,n]}(a_{is})
> $$
> 其中 $\alpha \in [0,1]$ 是用户设定的**注意力分数比例阈值**。

**通俗解释**：只要一个token的注意力分数达到"最大注意力分数的α倍"，就认为它是关键的，必须保留。

---

**定义 2：基于内积的关键 token**
> 给定所有键向量 $K = [k_1, \cdots, k_n]$，键 $k_j$ 是查询 $q_i$ 的关键token，当且仅当：
> $$
> q_i \cdot k_j^T \geq \max_{s \in [1,n]}(q_i \cdot k_s^T) - \beta
> $$

**定义3：DIPR查询（Dynamic Inner Product Range Query）**
> 给定键矩阵 $K$、查询向量 $q_i$ 和参数 $\beta \geq 0$，DIPR查询 $DIPR(q_i, \beta)$ 返回所有满足定义2的关键token集合 $cK$。

---
```
Algorithm 1: DIPRS(G, q, k0, l0, β)
Input: Graph G, query q, start key k0, capacity threshold l0, and β
Output: Critical token set cK
1 Initialize a list C with start key vector k0
2 i ← 0
3 while i < C.capacity() do
4   c_i ← the (i+1)-th key vector in C
5   i ← i + 1
6   foreach unvisited neighbor k of c_i in G do
7     tryAppend(q, k, β, C, l0)
8 ĉ ← the closest point to q in C
9 return cK ← {c | c ∈ C, q·c^T ≥ q·ĉ^T − β}

Procedure tryAppend(q, k, β, C, l0):
10 ĉ ← the closest point to q in C
11 Mark k as visited
12 if C.capacity() ≤ l0 or q·k^T ≥ q·ĉ^T − β then
13   C.append(k)
```

---

<!--
_class: title-page
header: HotPrefix: Hotness-Aware KV Cache Scheduling for Efficient Prefix Sharing in LLM Inference Systems
-->

## **HotPrefix**
### Efficient Prefix Sharing in LLM Inference Systems

---

prompt 工程（让 LLM 更好用的提示词技术）会导致 prompt 越来越长，大幅增加推理延迟、降低吞吐量。虽然前缀共享技术能复用相同开头的 KV 缓存，但存在致命问题：

- GPU 显存不够存储 KV 缓存
- 传统方法会使用 **最近最少使用** 策略转移缓存，导致**高热度前缀被淘汰**
- 转移的过程会产生大量的 IO 开销

---

### **HotPrefix**

HotPrefix 提出了三个创新点来解决这三个问题：

- 动态热度追踪
- 选择性 KV 缓存准入
- 热度提升

---

### **动态热度追踪**

论文用**热度感知布谷鸟过滤器**记录每个前缀树节点的热度信息，这是一种轻量级、高效的哈希表结构：
- 整体是连续地址空间的哈希表，包含n个桶，每个桶存4个条目。
- 每个条目存4个元数据：
  - 指纹（fingerprint）：前缀的紧凑唯一标识，用于快速查找。
  - 时钟（clock）：该前缀最后一次被加载到GPU后的时间
  - 频率（frequency）：该前缀被访问的总次数，直接反映热度。
  - 深度（depth）：该前缀在树中的位置（根节点深度为0）。

---

数据结构：增强型布谷鸟过滤器

![](./assets/cuckoo-filter.png)

---
当生成一个新的前缀节点时：
1. 提取从根节点到该节点的所有token作为唯一标识x。
2. 计算x的指纹f，再通过两个哈希函数计算两个候选桶：`b1=hash(f)`、`b2=b1⊕hash(f)`（⊕是异或运算）。
3. 如果两个桶中有空位置，就把f存入，同时初始化：clock=max_age（预定义最大值，论文实验设为255）、frequency=1、depth=节点当前深度。
4. 如果两个桶都满了，随机选一个桶的条目，把它移到它的备用桶；如果备用桶也满了，重复这个过程直到找到空位。
5. 如果迭代到最大次数还没找到空位，就把一些不用的条目移到CPU内存。

---

[步骤流程图](./assets/cuckoo-filter-insert.svg)

### 热度更新机制
分两步实时更新热度：
1. **节点被复用时**：找到该节点在布谷鸟过滤器中的条目，将frequency加1，同时把clock重置为max_age。
2. **周期性老化**：每处理固定数量的用户请求后，遍历整个布谷鸟过滤器，把所有条目的clock减1；如果clock降到0，就保持0不变。这个机制能区分"最近被访问的前缀"和"很久没被访问的前缀"。

---

### 热度感知驱逐策略
- **只驱逐叶子节点**（因为祖先节点被更多后代共享，热度更高）。
- 叶子节点的优先级计算公式：`priority = frequency + clock / length`
  - frequency：该节点的累计访问次数（即使被换出GPU再换入，次数也会累加）。
  - clock：该节点最后一次被加载到GPU后的时间。
  - length：该节点包含的token数量。
- **优先级越低的节点越先被驱逐**

---

### 选择性准入策略
不是所有被GPU驱逐的KV缓存都要移到CPU，论文观察到：很多低热度的KV缓存（比如用户特定的提问内容、LLM生成的独特内容）几乎不会被复用，存到CPU只会浪费I/O和内存。

因此采用**两步过滤**：
1. **频率阈值过滤**：如果被驱逐节点的frequency低于预定义阈值（论文实验设为10），直接丢弃。
2. **热度比较过滤**：通过频率阈值的节点，计算其热度：`hotness = frequency * clock`，然后和CPU内存中热度最低的节点比较。如果比CPU里最低的还低，直接丢弃；否则才移到CPU内存。

---

### 热度提升
如果等用户请求来了再从CPU加载KV缓存，传输开销可能比重新计算还大。因此HotPrefix采用**异步预取策略**，主动把CPU里的高热度KV缓存提前加载到GPU。

1. 把CPU内存中的节点按**热度降序**排序，把GPU内存中的叶子节点按**热度升序**排序。
2. 逐个处理CPU中的节点：
   - 如果该节点的父节点在GPU里，且已经被标记为要驱逐，直接跳过
   - 把该节点和GPU里最冷的节点比较替换
   - 如果一个GPU节点的空间不够，就选多个GPU节点

---

### 流水线提升与解码
1. 执行提升计划时，用**专门的CUDA流**异步传输KV缓存，和GPU的计算流并行。
2. 被替换的GPU节点直接丢弃（因为它们热度极低，复用概率几乎为0）。
3. 提升操作**不会增加GPU内存使用**，因为是等量替换。如果decode阶段需要更多内存，就用之前的热度感知驱逐策略释放空间。
