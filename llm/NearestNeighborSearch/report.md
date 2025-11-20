<!-- slide -->

In-Place Updates of a Graph Index for Streaming Approximate Nearest Neighbor Search

(流式近似最近邻搜索图索引的“原地”更新)





Haike Xu, et al.





arXiv 2025

<!-- slide vertical -->

<div class="box-centered">
<p class="title">研究背景 (1/2)</p>
<li class="text list"><strong>ANNS (近似最近邻搜索)</strong> 是现代信息系统的核心</li>
<li class="text list">应用：搜索引擎、推荐系统、RAG (检索增强生成)</li>
<li class="text list"><strong>邻近图 (Proximity Graph)</strong> 是目前最高效的 ANNS 索引</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title">研究背景 (2/2)</p>
<li class="text list">真实世界的数据是流式的：数据被高频地插入和删除</li>
<li class="text list"><strong>目标</strong>：ANNS 索引必须能实时处理这些更新，并保持高召回率</li>
<li class="text list">插入新节点相对容易，但删除非常困难</li>
</div>

<!-- slide -->

<div class="box-centered">
<p class="title">核心问题：为什么删除这么难？</p>
<li class="text list">图索引是单向链接的 (Singly-linked)</li>
<li class="text list">节点 p 只知道它的出邻居 $N_{out}(p)$</li>
<li class="text list">它不知道谁指向了它，即入邻居 $N_{in}(p)$</li>
<li class="text list">当删除 p 时，我们无法快速找到所有 $z \in N_{in}(p)$ 来修复断开的边</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>核心问题：为什么删除这么难？</p>
<li class="text list">图索引是单向链接的 (Singly-linked)</li>
<li class="text list">节点 p 只知道它的出邻居 $N_{out}(p)$</li>
<li class="text list">它不知道谁指向了它，即入邻居 $N_{in}(p)$</li>
<li class="text list">当删除 p 时，我们无法快速找到所有 $z \in N_{in}(p)$ 来修复断开的边</li>





<pre data-auto-animate><code>
z1 --> p --> w1
z2 --> p --> w2
z3 --> p
</code></pre>
<p class="text" data-auto-animate>我们想删除 p ...</p>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>核心问题：为什么删除这么难？</p>
<li class="text list">图索引是单向链接的 (Singly-linked)</li>
<li class="text list">节点 p 只知道它的出邻居 $N_{out}(p)$</li>
<li class="text list">它不知道谁指向了它，即入邻居 $N_{in}(p)$</li>
<li class="text list">当删除 p 时，我们无法快速找到所有 $z \in N_{in}(p)$ 来修复断开的边</li>





<pre data-auto-animate><code>
z1 --?   w1
z2 --?   w2
z3 --?
</code></pre>
<p class="text" data-auto-animate>删除 p 之后，z1, z2, z3 的边悬空了！</p>
<p class="text list"><strong>代价</strong>：牺牲内存，存储 $N_{in}(p)$ ？ -> 成本加倍，不可接受。</p>
</div>

<!-- slide -->

<div class="box-centered">
<p class="title">现有方案 (1/2)：软删除 (Soft Deletion)</p>
<li class="text list">给节点 p 贴上“墓碑” (Tombstone)，并不真删除</li>
<li class="text list">搜索时跳过带“墓碑”的节点</li>
<li class="text list"><strong>缺陷</strong>：图中“墓碑”越来越多，召回率 (Recall) 持续下降</li>
<li class="text list"><strong>最终</strong>：必须定期重建 (Rebuild) 整个索引</li>
<li class="text list list-2">重建十亿级索引：可能需要 2天 和 1.1TB 内存！</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title">现有方案 (2/2)：批量合并 (FreshDiskANN)</p>
<li class="text list">目前 SOTA (最先进) 的方法</li>
<li class="text list">将更新（增/删）暂存到“分段” (Segments) 中</li>
<li class="text list">查询时需要搜索多个“分段”，效率较低</li>
<li class="text list"><strong>核心缺陷</strong>：定期在后台进行**“重型合并” (Consolidation)</li>
<li class="text list list-2">合并操作会产生巨大的 CPU 和 IO 资源尖峰 (Spike)</li>
<li class="text list list-2">导致查询延迟飙升**，云服务成本剧增</li>
</div>

<!-- slide -->

<div class="box-centered">
<p class="title">本文方案：IP-DISKANN</p>
<li class="text list">IP = In-Place (原地)</li>
<li class="text list">第一个不需要批量合并的算法</li>
<li class="text list">高效地在**“原地”**处理每一次插入和删除</li>
<p class="text"><strong>目标：</strong> 避免资源尖峰，保持召回率稳定，实现真正的高吞吐流式更新。</p>
</div>

<!-- slide -->

<p class="title">IP-DISKANN 核心算法</p>
<p class="text">(Algorithm 5)</p>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>如何“原地”删除 p ?</p>
<p class="text">挑战：我们不知道 $N_{in}(p)$</p>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>如何“原地”删除 p ?</p>
<p class="text">挑战：我们不知道 $N_{in}(p)$</p>
<p class="text"><strong>Step 1: “近似” $N_{in}(p)$</strong></p>
<li class="text list">运行一次 GreedySearch 搜索 p 自己</li>
<li class="text list">搜索返回 Visited (已访问列表) 和 Candidates (候选列表)</li>
<li class="text list"><strong>$N&#39;_{in}(p)$</strong> = { Visited 中真正有边指向 p 的节点 z }</li>
<li class="text list">这是一个非常好的近似！</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>如何“原地”删除 p ?</p>
<p class="text"><strong>Step 2: 修复“上游” (z -> p)</strong></p>
<li class="text list">对于每一个近似入邻居 $z \in N&#39;_{in}(p)$：</li>
<li class="text list list-2">边 (z, p) 断了</li>
<li class="text list list-2">从 Candidates (k=50) 中，找到离 z 最近的 c 个节点 (c=3)</li>
<li class="text list list-2">添加 c 条新边：z -&gt; c 个新邻居</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>如何“原地”删除 p ?</p>
<p class="text"><strong>Step 3: 修复“下游” (p -> w)</strong></p>
<li class="text list">对于 p 的每一个出邻居 $w \in N_{out}(p)$：</li>
<li class="text list list-2">边 (p, w) 也断了</li>
<li class="text list list-2">从 Candidates (k=50) 中，找到离 w 最近的 c 个节点 (c=3)</li>
<li class="text list list-2">添加 c 条新边：c 个新邻居 -> w</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
<p class="title" data-auto-animate>如何“原地”删除 p ?</p>
<p class="text"><strong>Step 4: “轻量级”合并 (Algorithm 6)</strong></p>
<li class="text list">问题：$N&#39;_{in}(p)$ 只是近似，仍有未被发现的“悬空边”</li>
<li class="text list">IP-DISKANN 仍需周期性合并，但完全不同：</li>
<li class="text list list-2">FreshDiskANN (重型)：遍历 + 添加 $O(R^2)$ 条边 + 距离计算 + RobustPrune</li>
<li class="text list list-2"><strong>IP-DISKANN (轻型)</strong>：遍历 + 移除悬空ID</li>
<li class="text list"><strong>结论</strong>：新的合并只是内存清理，没有昂贵的距离计算！</li>
</div>

<!-- slide -->

<div class="box-centered">
<p class="title">实验设计：三种“酷刑测试” (Runbooks)</p>
<li class="text list"><strong>1. SlidingWindow (滑动窗口)</strong></li>
<li class="text list list-2">模拟“新闻索引”，新数据加入，旧数据删除</li>
<li class="text list"><strong>2. ExpirationTime (过期时间)</strong></li>
<li class="text list list-2">模拟数据有不同生命周期（永久, 长期, 短期）</li>
<li class="text list"><strong>3. Clustered (集群)</strong></li>
<li class="text list list-2">最难的测试。一次性插入或删除一整个数据簇</li>
</div>

<!-- slide -->

<p class="title">实验结果 (1/3)：召回率与查询速度</p>
<p class="text">在所有 Runbook 上 (Fig 1):</p>
<li class="text list">IP-DISKANN 的召回率 (Recall) 稳定且更高</li>
<li class="text list">IP-DISKANN 的查询速度 (QPS) 显著更高</li>
<li class="text list"><strong>为什么？</strong> 算法5 (原地) 比 算法4 (重型) 增加了更少、更有效的边，图的“导航性”更强</li>

<!-- slide vertical -->

<p class="title">实验结果 (2/3)：更新成本</p>
<p class="text">在大型数据集上 (Table 1):</p>
<li class="text list">IP-DISKANN 的总删除时间（包括轻量级合并）更少</li>
<li class="text list">HNSW 的更新成本高出 5-10 倍</li>
<li class="text list">IP-DISKANN (搜索密集型, 次线性) vs FreshDiskANN (扫描密集型, 线性)，规模越大，IP-DISKANN 优势越明显</li>

<!-- slide vertical -->

<p class="title">实验结果 (3/3)：惊人发现</p>
<p class="text">在最难的 Clustered 测试上 (Fig 2):</p>
<li class="text list">IP-DISKANN 和 FreshDiskANN 的召回率</li>
<li class="text list">竟然高于</li>
<li class="text list">“静态重建” (Static DiskANN) (即每一步都从头构建完美索引)</li>
<p class="text"><strong>推论：</strong>图索引是“越用越好”的。持续的增删和修复（无论是重型还是轻型）会“磨合”图的连通性，使其比初始状态更易于导航。</p>

<!-- slide -->

<div class="box-centered">
<p class="title">消融研究：关键参数 c</p>
<li class="text list">c = 每条断边的新增“替代边”数量</li>
<li class="text list">c=1：召回率低 (91.0%)</li>
<li class="text list">c=2：召回率大幅提升 (92.2%)</li>
<li class="text list">c=3：“甜点值” (92.5%)</li>
<li class="text list">c=5：召回率提升不大 (92.8%)，但删除时间线性增加</li>
</div>

<!-- slide -->

<div class="box-centered">
<p class="title">结论</p>
<li class="text list">IP-DISKANN 是第一个在单向链接图上实现原地删除的算法</li>
<li class="text list">它通过**“近似”+“定向修复”+“轻量级清理”三步实现</li>
<li class="text list"><strong>最大贡献：</strong> 彻底避免了批量合并的资源尖峰和查询抖动</li>
<li class="text list"><strong>结果：</strong> 实现了更高**的召回率、更快的查询速度 和 (at scale) 更低的更新成本</li>
<li class="text list">为 RAG 和搜索引擎提供了稳定、高效、低成本的流式更新方案</li>
</div>
