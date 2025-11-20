<style>
html {
  font-family: Consolas;
}
.box-centered {
  width: fit-content;
  margin: auto;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.box-left {
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
}
.text {
  font-size: 1.5rem;
  text-align: left;
  margin-top: 0.8rem !important;
  margin-bottom: 0.8rem !important;
}
.list {
  margin-left: 3rem !important;
  margin-top: 0.4rem !important;
  margin-bottom: 0.4rem !important;
}
.list-2 {
  margin-left: 5rem !important
}
.title {
  font-size: 2.8rem;
  text-align: left;
  font-weight: bold;
}
.font-larger {
  font-size: 2rem;
}
.img-tip {
  font-size: 1.32rem;
  text-align: center;
  color: rgba(255, 255, 255, 0.45);
  margin-top: 1rem !important;
}
.img-fit {
  margin-bottom: 0 !important;
  max-height: 45vh !important;
}
.img-right {
  max-height: 55vh !important;
  max-width: 32vw !important
}
.box-four-combo {
  margin-top: 4rem;
  display: flex;
  flex-direction: row;
  justify-content: center;
  width: 100%;
}
.four-combo {
  width: 18%;
  margin-left: 2rem !important;
  margin-right: 2rem !important;
}
.reveal pre code {
  font-family: Consolas;
  font-size: 1.2rem;
  line-height: 2rem;
  overflow: visible;
  padding-right: 1.5rem;
}
</style>

<!-- slide -->

**CacheBlend: Fast Large Language Model Serving for RAG with Cached Knowledge Fusion**
EuroSys 2025

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">问题发现</p>
  <li class="text list">KV Cache 只能对完整的输入前缀进行缓存</li>
  <li class="text list">如果在中间插入独立的文本块，它的 KV Cache 不包含对前面文本信息的注意力</li>
  <li class="text list">尤其是在 RAG 应用中，一次提问还需要另外包含几个从向量库中检索的文本块</li>
  <p class="title">本文目标</p>
  <li class="text list">当一个 LLM 输入包含多个文本块时，如何快速地组合它们各自预先算好的 KV 缓存</li>
  <li class="text list">同时保证生成质量</li>
</div>

<!-- slide -->

**选择性重计算**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">为什么不需要重算所有词？</p>
  <li class="text list">随着“重计算比例 (%)”增加，“前向注意力偏差”会下降</li>
  <li class="text list">最大的降幅发生在重计算比例刚开始增加时</li>
  <li class="text list">在注意力矩阵中，高注意力的连接通常只发生在少数词之间</li>
  <p class="title" style="font-size: 2rem;">如何在没有完整 KV 的情况下找到 KV 偏差高的词？</p>
  <li class="text list">在第 i 层上 KV 偏差最高的词，极有可能在第 i+1 层上 KV 偏差也最高</li>
  <li class="text list">Transformer 中，词的嵌入在层间变化缓慢 ，而 KV 缓存是由嵌入线性转换而来的</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">渐进式过滤 (Gradual Filtering)</p>   <li class="text list">仅使用第 1 层的 HKVD 词在深层中会不准确 </li>
  <li class="text list">CacheBlend 采用逐层过滤的方案 ：</li>
  <li class="text list list-2">第 1 层： 完全重算，选出 r1​% (例如 20%) 偏差最高的词 </li>
  <li class="text list list-2">第 2 层： 仅为这 r1​% 的词重算，并从它们之中选出 r2​% (例如 18%) </li>
  <li class="text list list-2">第 i 层： 重复此过程，逐步筛选出在多层上偏差都高的词，比例逐渐收敛到目标比例 (例如 15%) </li>
  <li class="text list">此方案能更可靠地在每一层识别出 HKVD 词 </li>
</div>

<!-- slide vertical -->

**管线化设计（Pipeline）**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">延迟隐藏 (Latency Hiding)</p>   <li class="text list">"选择性重计算" 仍有计算开销 (T_recompute) </li>
  <li class="text list">"加载 KV 缓存" 也有 I/O 开销 (T_load)，尤其当缓存存储在 SSD 上时 </li>
  <li class="text list">核心洞察： 这两个操作可以管线化执行 </li>
  <li class="text list list-2">在 GPU 计算 第 i 层的重计算时 (T_recompute)...</li>
  <li class="text list list-2">...CPU 同时 从 SSD 加载 第 i+1 层的 KV 缓存 (T_load) </li>
  <li class="text list">只要 Trecompute​≤Tload​，重计算的延迟就完全被隐藏 </li>
  <li class="text list">结果： 允许将 KV 缓存存储在更慢、更便宜、容量更大的设备 (如 SSD) 上，而不增加 TTFT </li>
</div>

<!-- slide -->

**系统架构**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title" style="font-size: 2.3rem;">CacheBlend 的三个关键组件</p>   <li class="text list">1. 加载控制器 (Loading Controller) </li>
  <li class="text list list-2">决策者。估算 Tload​ 和 Trecompute​ </li>
  <li class="text list list-2">决定最佳重计算比例 r% 和存储策略 </li>
  <li class="text list">2. KV 缓存存储 (KV Cache Store) </li>
  <li class="text list list-2">通过哈希管理文本块的 KV 缓存 </li>
  <li class="text list list-2">存储在 CPU 内存或 SSD 上，使用 LRU 淘汰 </li>
  <li class="text list">3. 融合器 (Fusor) </li>
  <li class="text list list-2">执行者。在 GPU 上执行“渐进式过滤” </li>
  <li class="text list list-2">按管线化调度执行计算和加载 </li>
</div>
