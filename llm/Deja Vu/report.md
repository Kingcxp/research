<style> html { font-family: Consolas; } .box-centered { width: fit-content; margin: auto; display: flex; flex-direction: column; justify-content: center; } .box-left { flex: 1; height: 100%; display: flex; flex-direction: column; } .text { font-size: 1.5rem; text-align: left; margin-top: 0.8rem !important; margin-bottom: 0.8rem !important; } .list { margin-left: 3rem !important; margin-top: 0.4rem !important; margin-bottom: 0.4rem !important; } .list-2 { margin-left: 5rem !important } .title { font-size: 2.8rem; text-align: left; font-weight: bold; } .font-larger { font-size: 2rem; } .img-tip { font-size: 1.32rem; text-align: center; color: rgba(255, 255, 255, 0.45); margin-top: 1rem !important; } .img-fit { margin-bottom: 0 !important; max-height: 45vh !important; } .img-right { max-height: 55vh !important; max-width: 32vw !important } .box-four-combo { margin-top: 4rem; display: flex; flex-direction: row; justify-content: center; width: 100%; } .four-combo { width: 18%; margin-left: 2rem !important; margin-right: 2rem !important; } .reveal pre code { font-family: Consolas; font-size: 1.2rem; line-height: 2rem; overflow: visible; padding-right: 1.5rem; } </style>

<!-- slide  -->

**Deja Vu: Contextual Sparsity for Efficient LLMs at Inference Time**

ICML 2023

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">问题发现</p>
  <li class="text list">LLM（大语言模型）在推理时计算成本极高，延迟是瓶颈</li>
  <li class="text list">稀疏性 (Sparsity) 是自然解决办法，但现有方法存在三大缺陷：</li>
  <li class="text list list-2">(1) 需要昂贵的重训练 (Retraining)</li>
  <li class="text list list-2">(2) 损害模型的上下文学习 (In-Context Learning) 能力</li>
  <li class="text list list-2">(3) 无法在GPU上实现真正的 "wall-clock" 加速</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">论文思考</p>
  <li class="text list">本文假设存在一种 上下文稀疏性 (Contextual Sparsity)</li>
  <li class="text list list-2">即：对于特定输入，模型中只激活一个很小的、依赖于输入的参数子集</li>
  <li class="text list">本文的目标是证明：</li>
  <li class="text list list-2">1. 这种稀疏性存在</li>
  <li class="text list list-2">2. 这种稀疏性可以被低成本地预测</li>
  <li class="text list list-2">3. 这种稀疏性可以被利用来真正加速（Wall-clock time）</li>
</div>

<!-- slide -->

**核心概念：上下文稀疏性**

<!-- slide vertical data-auto-animate -->

<div class="box-centered" data-auto-animate>
  <p class="title">什么是上下文稀疏性？</p>
  <li class="text list">静态稀疏 (Static Sparsity)：(如剪枝)</li>
  <li class="text list list-2">模型中某些参数 永远 是0</li>
  <li class="text list list-2">对所有输入都使用同一套稀疏参数</li>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered" data-auto-animate>
  <p class="title">什么是上下文稀疏性？</p>
  <li class="text list">静态稀疏 (Static Sparsity)：(如剪枝)</li>
  <li class="text list list-2">模型中某些参数 永远 是0</li>
  <li class="text list list-2">对所有输入都使用同一套稀疏参数</li>
  <li class="text list">上下文稀疏 (Contextual Sparsity)：(DEJAVU)</li>
  <li class="text list list-2">稀疏模式是动态的、依赖于输入的</li>
  <li class="text list list-2">输入A -> 激活 {Head 1, 3, 5}, {MLP 2, 4, 6}</li>
  <li class="text list list-2">输入B -> 激活 {Head 2, 4, 6}, {MLP 1, 3, 5}</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">验证：稀疏性真的存在吗？</p>
  <li class="text list">论文使用 "双通道" (Two-Pass) 方法验证：</li>
  <li class="text list list-2">第一遍：正常（稠密）运行，记录下产生“大输出范数”的Attention头和MLP神经元</li>
  <li class="text list list-2">第二遍：只使用第一遍记录的 "重要" 参数运行</li>
  <li class="text list">惊人发现：第二遍（稀疏）的性能和第一遍（稠密）几乎完全相同</li>
  <li class="text list">平均而言，模型存在 85% 的结构化稀疏：</li>
  <li class="text list list-2">~80% 的 Attention 头可以被关闭</li>
  <li class="text list list-2">~95% 的 MLP 神经元可以被关闭</li>
</div>

<!-- slide vertical -->

**理论洞察：为什么LLM是稀疏的？**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">洞察 1：Attention是一种聚类</p>
  <li class="text list">观察：不同的头有不同作用</li>
  <li class="text list list-2">"重击者" (Heavy Hitter) 头：高度关注 "shipping", "like" 等关键Token</li>
  <li class="text list list-2">"均匀混合" (Uniform) 头：对所有Token的注意力是分散、模糊的</li>
  <li class="text list">假说：Attention 的计算过程在数学上类似于 "均值漂移聚类" (Mean-Shift Clustering) 的一步迭代</li>
  <li class="text list list-2">"重击者"的头找到了有意义的 "簇"（贡献大）</li>
  <li class="text list list-2">"均匀"的头没有找到 "簇"（贡献小，可被稀疏）</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">洞察 2：跨层嵌入缓慢变化</p>
  <li class="text list">观察：同一个Token在第 l 层和第 l+1 层的嵌入向量，其余弦相似度高达 0.99</li>
  <li class="text list">原因：残差连接 Output=X+F(X)</li>
  <li class="text list list-2">论文发现，输入 X 的范数 ∣∣X∣∣ 远大于 F(X)（Attention或MLP计算结果）的范数 ∣∣F(X)∣∣</li>
  <li class="text list list-2">F(X) 只是对 X 的一个微小修正，因此 Output≈X</li>
  <li class="text list">至关重要的推论：</li>
  <li class="text list list-2">既然第 l 层的输入 yl​ 和 l+1 层的输入 yl+1​ 几乎一样，我们就可以用 yl​ 来提前预测 yl+1​ 的稀疏模式</li>
</div>

<!-- slide -->

**DEJAVU 系统设计**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">挑战 1：稀疏性预测</p>
  <li class="text list">目标：必须在 执行 计算 之前 预测出稀疏模式</li>
  <li class="text list">建模：一个 "近似最大内积" (Approx-MaxIP) 问题，本质是最近邻搜索 (NNS)</li>
  <li class="text list">难点：标准 NNS 库 (HNSW, FAISS) 太慢了</li>
  <li class="text list list-2">HNSW 预测需 10ms</li>
  <li class="text list list-2">MLP 计算（稠密）仅需 0.2ms</li>
  <li class="text list list-2">预测开销 > 计算收益，不可接受</li>
  <li class="text list">DEJAVU 方案：</li>
  <li class="text list list-2">使用一个小型的神经网络 (NN) 分类器作为预测器，它可以在 GPU 上快速执行</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">挑战 2：效率 (预测开销)</p>
  <li class="text list">常规串行执行 (慢)：</li>
  <li class="text list list-2">[预测 MHAl​] → [计算 MHAl​] → [预测 MLPl​] → [计算 MLPl​]</li>
  <li class="text list list-2">预测的延迟被累加到总延迟中</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">挑战 2：效率 (预测开销)</p>
  <li class="text list">常规串行执行 (慢)：</li>
  <li class="text list list-2">[预测 MHAl​] → [计算 MHAl​] → [预测 MLPl​] → [计算 MLPl​]</li>
  <li class="text list list-2">预测的延迟被累加到总延迟中</li>
  <li class="text list">DEJAVU 异步前瞻预测 (快)：(利用 "嵌入缓慢变化")</li>
  <li class="text list list-2">主线程：... [计算 MHAl​] ... → ... [计算 MLPl​] ...</li>
  <li class="text list list-2">预测线程：... [预测 MLPl​] ... → ... [预测 MHAl+1​] ...</li>
  <li class="text list list-2">(使用 MHAl​ 的输入 yl​ 来预测 MLPl​；使用 MLPl​ 的输入 yl​~​ 来预测 MHAl+1​)</li>
  <li class="text list">效果：预测开销被主计算的延迟完美隐藏</li>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">挑战 2：效率 (硬件感知)</p>
  <li class="text list">难题：为什么标准稀疏在GPU上很慢？</li>
  <li class="text list list-2">W[idx,:] * y (PyTorch稀疏) 需要 3x 内存I/O：(1)读索引 → (2)读非连续数据并写回连续内存 → (3)读连续数据并计算</li>
  <li class="text list list-2">I/O 成为瓶颈，比稠密还慢</li>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">挑战 2：效率 (硬件感知)</p>
  <li class="text list">难题：为什么标准稀疏在GPU上很慢？</li>
  <li class="text list list-2">W[idx,:] * y (PyTorch稀疏) 需要 3x 内存I/O：(1)读索引 → (2)读非连续数据并写回连续内存 → (3)读连续数据并计算</li>
  <li class="text list list-2">I/O 成为瓶颈，比稠密还慢</li>
  <li class="text list">DEJAVU 硬件优化 (快)：</li>
  <li class="text list list-2">1. 内核融合 (Kernel Fusion)：</li>
  <li class="text list list-2">使用 Triton 编写自定义GPU内核，将 "索引" 和 "计算" 融合，I/O开销从 3x 降到 1x。比 PyTorch 快 4-5 倍。</li>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">挑战 2：效率 (硬件感知)</p>
  <li class="text list">难题：为什么标准稀疏在GPU上很慢？</li>
  <li class="text list list-2">W[idx,:] * y (PyTorch稀疏) 需要 3x 内存I/O：(1)读索引 → (2)读非连续数据并写回连续内存 → (3)读连续数据并计算</li>
  <li class="text list list-2">I/O 成为瓶颈，比稠密还慢</li>
  <li class="text list">DEJAVU 硬件优化 (快)：</li>
  <li class="text list list-2">1. 内核融合 (Kernel Fusion)：</li>
  <li class="text list list-2">使用 Triton 编写自定义GPU内核，将 "索引" 和 "计算" 融合，I/O开销从 3x 降到 1x。比 PyTorch 快 4-5 倍。</li>
  <li class="text list list-2">2. 内存合并 (Memory Coalescing)：</li>
  <li class="text list list-2">问题： W2 和 WO 需要按列索引，内存不连续。</li>
  <li class="text list list-2">技巧：加载模型时，预先转置 (Transpose) W2 和 WO 矩阵。使 "列索引" 变为 "行索引"，实现连续内存访问。</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">挑战 3：KV Cache 处理</p>
  <li class="text list">问题：如果T时刻跳过了一个Head，T+1时刻这个Head被激活时，KV Cache中就缺少T时刻的K和V</li>
  <li class="text list">DEJAVU 方案 (I/O与计算的权衡)：</li>
  <li class="text list list-2">对于被选中 (Active) 的头：正常计算 K, V 并存入 Cache</li>
  <li class="text list list-2">对于被跳过 (Inactive) 的头：不计算 K, V。作为替代，保存当前Token的输入嵌入 y</li>
  <li class="text list list-2">未来如果需要：当这个头被激活时，"按需" (on-the-fly) 加载 y 并即时计算出缺失的 K, V</li>
  <li class="text list">为什么这很快？：LLM推理是 I/O 瓶颈。该方法用少量 "免费" 的计算，换取了海量的 "昂贵" 的 I/O 节省（因为不需要加载 WK​,WV​ 矩阵）。</li>
</div>
