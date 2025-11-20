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

**PagedAttention: Efficient Memory Management for Large Language Model Serving with PagedAttention**

SOSP 2023

<!-- slide vertical -->

<div class="box-centered">
  <p class="title" data-auto-animate>问题的核心：内存</p>
  <li class="text list">LLM 服务吞吐量受限于<b>批处理 (Batching)</b> 大小</li>
  <li class="text list">批处理大小受限于 <b>KV 缓存</b> 内存</li>
  <li class="text list">KV 缓存 (a) 巨大 (b) 动态增长 (c) 长度未知</li>
  <li class="text list">内存管理是 LLM 服务的<b>核心瓶颈</b></li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title" data-auto-animate>现有系统的“原罪”：连续内存</p>
  <li class="text list">现有系统 (如 FasterTransformer, Orca)</li>
  <li class="text list">要求为每个请求分配<b>连续的</b>内存块</li>
  <li class="text list">必须<b>预先分配</b>一个最大长度 (如 2048) 的空间</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title" data-auto-animate>现有系统的“原罪”：连续内存</p>
  <p class="text" style="color: #ff6363; font-weight: bold;">这导致了灾难性的后果：</p>
  <li class="text list"><b>1. 严重的内存浪费 (碎片化)</b></li>
  <li class="text list list-2"><b>内部碎片</b>：预留了 2048，只用了 100</li>
  <li class="text list list-2"><b>外部碎片</b>：内存中存在大量小的、不连续的空闲块</li>
  <li class="text list"><b>2. 无法实现内存共享</b></li>
  <li class="text list list-2">并行采样、束搜索、共享前缀...</li>
  <li class="text list list-2">相同的 Prompt 的 KV 缓存必须被<b>复制多份</b></li>
</div>

<!-- slide -->

解决方案：PagedAttention

<!-- slide vertical -->

<div class="box-centered">
  <p class="title" data-auto-animate>核心思想：来自 OS 的古老智慧</p>
  <li class="text list"><b>灵感来源</b>：操作系统的<b>虚拟内存 (Virtual Memory)</b> 和<b>分页 (Paging)</b></li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title" data-auto-animate>核心思想：来自 OS 的古老智慧</p>
  <li class="text list"><b>OS 的工作</b>:
<span style="color: #1c73b5ff">程序逻辑页 (Logical Page)</span> ->
<span style="color: #006d00ff">物理内存页 (Physical Page)</span>
</li>
  <li class="text list"><b>vLLM 的工作</b>:
<span style="color: #1c73b5ff">请求逻辑块 (Logical Block)</span> ->
<span style="color: #006d00ff">GPU 物理块 (Physical Block)</span>
</li>





  <p class="text" style="font-weight: bold; font-size: 2rem; text-align: center;"><b>放弃连续内存！</b></p>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">PagedAttention: 块表 (Block Table)</p>
  <li class="text list">将 KV 缓存分割成<b>固定大小的“块” (Block)</b></li>
  <li class="text list">块在物理内存中<b>无需连续</b></li>
  <li class="text list">使用<b>“块表” (Block Table)</b> 来管理映射关系</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">vLLM 工作流程</p>
  <li class="text list" style="font-size: 1.2rem;"><b>提示阶段</b>：分配 Prompt 所需的块 (逻辑 0, 1 -> 物理 7, 1)</li>
  <li class="text list" style="font-size: 1.2rem;"><b>生成第 1 步</b>：新 Token 填入 Block 1 的空槽 (物理块 1)</li>
  <li class="text list" style="font-size: 1.2rem;"><b>生成第 2 步</b>：Block 1 已满。<b>按需分配</b>新块 (逻辑 2 -> 物理 3)</li>
  <li class="text list" style="font-size: 1.2rem; color: #006d00ff;"><b>结论</b>：内存浪费 (内部碎片) 仅限于最后一个块</li>
</div>

<!-- slide -->

PagedAttention 的“超能力”：内存共享

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">特性1：写时复制 (Copy-on-Write)</p>
  <li class="text list" style="font-size: 1.2rem;"><b>场景</b>：并行采样 (1 个 Prompt, 2 个输出 A1, A2)</li>
  <li class="text list" style="font-size: 1.2rem;"><b>共享</b>：A1 和 A2 的块表指向<b>相同</b>的物理块 (引用计数=2)</li>
  <li class="text list" style="font-size: 1.2rem;"><b>写入时</b>：当 A1 尝试写入共享块时，系统<b>复制</b>一个新块 (物理块 3) 供 A1 使用</li>
  <li class="text list" style="font-size: 1.2rem; color: #006d00ff;"><b>结果</b>：极低的复制开销 (仅一个块)，节省了长 Prompt 的大量内存</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">特性2：高效的束搜索 (Beam Search)</p>
  <li class="text list">PagedAttention 天然支持 Beam Search 的<b>树状共享</b>结构</li>
  <li class="text list"><b>无需内存复制</b>：候选者被剪枝时，只需降低物理块的引用计数</li>
  <li class="text list">引用计数为 0 的块被立即释放</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">特性3：共享前缀 / 系统提示</p>
  <li class="text list">服务商可以<b>预先缓存</b>共享前缀 (如系统提示) 的 KV 块</li>
  <li class="text list">所有新请求可以直接映射到这些<b>已缓存</b>的物理块</li>
  <li class="text list">极大加快了<b>提示阶段 (Prefill)</b> 的速度</li>
</div>
