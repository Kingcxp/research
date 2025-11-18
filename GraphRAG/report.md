<style>
html {
  font-family: Consolas;
}
.box-centered {
  width: 80%;
  margin: auto;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
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

**From Local to Global: A GraphRAG Approach to Query-Focused Summarization**
arXiv:2404.16130v2 (Microsoft Research)

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">问题发现</p>
  <li class="text list">传统 RAG (Vector RAG) 擅长回答**局部问题** (Local Questions)。</li>
  <li class="text list">例如："NeoChip 是哪年被收购的？"</li>
  <li class="text list">但它无法回答针对**整个**文档库的**全局性问题** (Global Questions)。</li>
  <li class="text list">例如："这个数据集中有哪些主要主题？" 或 "反复出现的争议点是什么？"</li>
  <p class="title">论文思考</p>
  <li class="text list">全局问题本质上是“查询导向的摘要”(QFS) 任务，而非“检索”任务。</li>
  <li class="text list">传统 RAG 检索的几个孤立片段 <b>≠</b> 全局概括。</li>
  <li class="text list">因此，本论文提出 <strong>GraphRAG</strong>，一种通过图结构预先捕捉全局信息的方法。</li>
</div>

<!-- slide -->

**GraphRAG 核心流程**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">GraphRAG 核心流程</p>
  <li class="text">GraphRAG 分为两个阶段：</li>

  <p class="text font-larger" style="margin-top: 2rem !important;"><b>阶段一：索引时间 (Indexing Time)</b> <span style="font-size: 1.2rem;">(用户提问前完成)</span></p>
  <li class="text list list-2">1. 源文档 -> 2. 文本块</li>
  <li class="text list list-2">3. (LLM) -> 实体与关系</li>
  <li class="text list list-2">4. -> 知识图谱 (Knowledge Graph)</li>
  <li class="text list list-2">5. (社区检测) -> 图社区 (Graph Communities)</li>
  <li class="text list list-2">6. (LLM) -> 社区摘要 (Community Summaries)</li>

  <p class="text font-larger" style="margin-top: 2rem !important;"><b>阶段二：查询时间 (Query Time)</b> <span style="font-size: 1.2rem;">(用户提问时)</span></p>
  <li class="text list list-2">1. (问题 + 社区摘要) -> (LLM Map)</li>
  <li class="text list list-2">2. -> 社区答案 (局部答案)</li>
  <li class="text list list-2">3. -> (LLM Reduce)</li>
  <li class="text list list-2">4. -> 全局答案 (Global Answer)</li>
</div>

<!-- slide -->

**索引阶段：构建分层摘要**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">步骤 1 & 2：文本 -> 实体与关系</p>
  <li class="text">LLM 读取每个文本块，提取 实体(Nodes)、关系(Edges) 和 声明(Claims)。</li>
  <li class="text"><b>原始文本 (示例):</b></li>
  <pre><code>"NeoChip (NC) ... 之前是一家私营实体，
在 2016 年被 Quantum Systems 收购。"</code></pre>
  <li class="text"><b>LLM 提取:</b></li>
  <li class="text list list-2"><b>实体:</b> NeoChip (描述: "上市公司，专攻低功耗处理器...")</li>
  <li class="text list list-2"><b>实体:</b> Quantum Systems (描述: "曾经拥有 NeoChip 的公司...")</li>
  <li class="text list list-2"><b>关系:</b> NeoChip --[被收购]-- Quantum Systems (描述: "2016年...")</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">步骤 3 & 4：图谱 -> 社区</p>
  <li class="text" style="opacity: 0.8;">LLM 读取每个文本块，提取 实体(Nodes)、关系(Edges) 和 声明(Claims)。</li>
  <li class="text" style="opacity: 0.8;"><b>原始文本 (示例):</b> ... </li>
  <li class="text" style="opacity: 0.8;"><b>LLM 提取:</b> ... </li>
  <br>
  <li class="text">所有提取的 实体/关系 被汇聚成一个大规模的**知识图谱**。</li>
  <li class="text">使用**社区检测** (e.g., Leiden 算法) 对图谱进行**分层**。</li>
  <li class="text list"><b>C0 (根社区)</b> (e.g., "科技", "金融")</li>
  <li class="text list list-2"><b>C1 (子社区)</b> (e.g., "AI", "硬件", "软件")</li>
  <li class="text list list-2" style="margin-left: 7rem !important"><b>C2 (叶社区)</b> (e.g., "芯片", "GPU")</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">步骤 5：社区 -> 摘要</p>
  <li class="text" style="opacity: 0.8;">LLM 读取每个文本块...</li>
  <li class="text" style="opacity: 0.8;">所有提取的 实体/关系 被汇聚成一个大规模的<strong>知识图谱</strong>。</li>
  <li class="text" style="opacity: 0.8;">使用<strong>社区检测</strong> (e.g., Leiden 算法) 对图谱进行<strong>分层</strong>。</li>
  <li class="text" style="opacity: 0.8;"><b>C0 (根社区)</b> ...</li>
  <br>
  <li class="text">LLM <strong>自下而上</strong> (Bottom-up) 生成“社区摘要”。</li>
  <li class="text list list-2">`Summary(C2)` = LLM 总结 C2 社区内的实体和关系。</li>
  <li class="text list list-2">`Summary(C1)` = LLM 递归地总结 `(Summary(C2_a) + Summary(C2_b) ...)`</li>
  <li class="text list list-2">`Summary(C0)` = LLM 递归地总结 `(Summary(C1_a) + Summary(C1_b) ...)`</li>
  <li class="text"><br><b>索引完成！</b> 得到一个分层的、涵盖全局到局部的摘要库。</li>
</div>

<!-- slide -->

**查询阶段：Map-Reduce 摘要**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">算法流程 (Query Time)</p>
  <li class="text">用户提问: "数据集中关于隐私法的观点有哪些？"</li>
  <li class="text">1. <b>选择摘要层级</b> (e.g., C2 摘要库)</li>
  <li class="text">2. <b>Map (并行)</b>：将问题分发给<b>所有</b> C2 摘要</li>
  <li class="text list list-2">`[问题]` + `[C2_a 摘要]` -> (LLM) -> `[社区答案 A]` + `[分数: 90]`</li>
  <li class="text list list-2">`[问题]` + `[C2_b 摘要]` -> (LLM) -> `[社区答案 B]` + `[分数: 20]`</li>
  <li class="text list list-2">`[问题]` + `[C2_c 摘要]` -> (LLM) -> `[社区答案 C]` + `[分数: 75]`</li>
  <li class="text">3. <b>Reduce (汇总)</b>：</li>
  <li class="text list list-2">按分数排序 (A, C, B...)</li>
  <li class="text list list-2">`[问题]` + `[答案 A]` + `[答案 C]` ... (塞满上下文) -> (LLM)</li>
  <li class="text">4. <b>返回最终的“全局答案”</b></li>
</div>

<!-- slide -->

**实验结果**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">结果：LLM 评委 (图 2)</p>
  <li class="text"><b>GraphRAG vs 传统 RAG (SS)</b></li>
  <li class="text list list-2">GraphRAG (C0-C3) 在 **全面性** 和 **多样性** 上 **完胜** SS。</li>
  <li class="text list list-2">(e.g., 播客数据：全面性胜率 72-83%)</li>
  <li class="text list list-2">SS 只在 **直接性** 上获胜 (符合预期，答案更短但片面)。</li>
  <li class="text"><br><b>GraphRAG (C) vs 无图 (TS)</b></li>
  <li class="text list list-2">GraphRAG (C1-C3) **略微优于** TS。</li>
  <li class="text list list-2">证明了“图”结构（连接性）提供了比“无图”方法（TS）额外的好处。</li>
  <li class="text"><br><b>客观验证 (实验 2)</b></li>
  <li class="text list list-2">基于“事实声明”(Claims) 数量的客观统计也验证了：</li>
  <li class="text list list-2"><b>全局方法 (GraphRAG/TS) > 传统 RAG (SS)</b></li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">结果：效率的巨大优势 (表 2)</p>
  <li class="text">GraphRAG 不仅效果好，而且在查询时**效率极高**。</li>
  <p class="text" style="margin-top: 2rem !important;"><b>查询时需处理的 Token (播客数据)</b></p>
  <li class="text list"><b>TS (无图)</b>: 1,014,611 Tokens (100% - 完整数据)</li>
  <li class="text list"><b>C3 (低层级)</b>: 746,100 Tokens (73.5%)</li>
  <li class="text list"><b>C2 (中层级)</b>: 565,720 Tokens (55.8%)</li>
  <li class="text list"><b>C1 (高层级)</b>: 225,756 Tokens (22.2%)</li>
  <li class="text list"><b>C0 (根层级)</b>: <b style="color: #7FFF00;">26,657 Tokens (2.6%)</b></li>
  <li class="text"><br><b>结论：</b></li>
  <li class="text list">使用 C0 摘要库，性能依然远超 SS，但查询**成本降低了 97%**！</li>
  <li class="text list">非常适合需要快速、迭代探索的“意义构建”(sensemaking) 任务。</li>
</div>

<!-- slide -->

**结论**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">结论</p>
  <li class="text list">论文提出了 **GraphRAG**，一种通过知识图谱和分层摘要来解决 RAG **全局理解**难题的新方法。</li>
  <li class="text list">实验证明，GraphRAG 在**全面性**和**多样性**上显著优于传统 Vector RAG。</li>
  <li class="text list">GraphRAG（尤其是 C0 级别）在查询时表现出极高的**效率优势**（Token 成本降低 97%+）。</li>
  <li class="text list">论文还贡献了一种新颖的、针对“全局问题”的**评估方法** (自适应基准 + LLM 评委)。</li>
  <li class="text list" style="margin-top: 2rem !important;"><b>项目已开源 (github.com/microsoft/graphrag)</b></li>
</div>
