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
  font-size: 1.8rem;
  text-align: left;
  margin-top: 0.8rem !important;
  margin-bottom: 0.8rem !important;
}
.list {
  margin-left: 3rem !important;
  margin-top: 0.4rem !important;
  margin-bottom: 0.4rem !important;
}
.title {
  font-size: 2.8rem;
  text-align: left;
  font-weight: bold;
}
.font-larger {
  font-size: 2.4rem;
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

## Kernel GPT
#### 通过大型语言模型增强内核模糊测试
****
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;@Kingcq 2025

<!-- slide -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">模糊测试技术已经被使用了数十年</p>
  <p class="text">这些技术生成大量的系统调用作为测试用例</p>
  <p class="text">这些测试用例旨在检测可能存在的内核错误（崩溃、越界写入等 ）</p>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">在这些技术中，<strong>Syzkaller</strong> 是最流行的工具之一</p>
  <p class="text">它提供了一种叫做 <strong>syzlang</strong> 的语言，用以规范系统调用的生成过程</p>
  <p class="text">一条编写良好的 <strong>syzlang</strong> 能有效规范系统调用的语法及依赖关系</p>
  <p class="text">从而使其能够生成更有效的系统调用序列，测试能够更加深入</p>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">然而，编写这样一个完备的规范是十分困难的。</p>
  <p class="text">最简单的想法是使用静态分析技术配合人工编写规范。</p>
  <img class="img-fit" src="./assets/Figure_1_1.png" />
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">这种流程由专家手动定义规则开始，再经过静态分析工具总结规范</p>
  <p class="text">但这很大程度上依赖于专家对于现有代码库和 <strong>Syzkaller</strong> 的理解</p>
  <p class="text">此外，随着内核代码的演变，由专家定义的这些规则还需要频繁更新</p>
  <img class="img-fit" src="./assets/Figure_1_1.png" />
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">一些特殊情况的出现，就会导致错误的系统调用规范</p>
  <p class="text">比如下图中的数据结构</p>
  <img class="img-fit" src="./assets/Figure_2_ab.png" />
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">现有的系统调用规范生成器会使用 <strong>name</strong> 字段来确定驱动交互的设备名称</p>
  <p class="text">然而，在一种合法但少见的用例中</p>
  <p class="text">正确的设备名称是在 <strong>nodename</strong> 字段中指定的</p>
  <img class="img-fit" src="./assets/Figure_2_ab.png" />
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">• 模糊测试技术</p>
  <p class="text">关键在于，我们能否自动化并改进生成规则的过程，同时减少工作量？</p>
</div>

<!-- slide data-auto-animate -->

<div class="box-centered">
  <p class="title">• 使用 LLM</p>
  <p class="text">大语言模型在预训练过程中基础过相关的训练数据</p>
  <p class="text">其优越的知识库使它能分析代码并生成高质量、可读的规范</p>
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 使用 LLM</p>
  <p class="text">如图所示，我们可以自动化从代码库到 syscall 规范的规则推断过程</p>
  <p class="text">因为 LLM 本身可以分析代码，</p>
  <p class="text">这种方法消除了在复杂的静态分析工具中硬编码规则的需要</p>
  <p class="text">这大大简化了适应内核代码库变化的过程</p>
  <img class="img-fit" src="./assets/Figure_1_2.png" />
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• 使用 LLM</p>
  <p class="text">传统方法的限制</p>
  <p class="text list">• L-1：建模不完整。基于规则的方法难以捕捉内核代码模式的多样性，导致覆盖范围有限。维护这些规则具有挑战性且不切实际。</p>
  <p class="text list">• L-2：可读性。静态分析生成的规范通常难以理解，阻碍了验证和维护。</p>
  <p class="text list">• L-3：文本理解。这些工具难以从文本信息（如注释）中推断规范，限制了它们捕捉 syscall 行为的底层含义和意图的能力。</p>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">• 使用 LLM</p>
  <p class="text">利用 LLM 优势的新方法</p>
  <p class="text list">• 缓解 L-1：LLM 在大量代码库上进行预训练，使其能够比静态分析规则更有效地处理更广泛的情况。</p>
  <p class="text list">• 缓解 L-2：LLM 可以根据代码生成规范中的描述性和人类可读的名称，增强可读性和维护性。</p>
  <p class="text list">• 缓解 L-3：LLM 擅长解释文本信息，生成能够捕捉 syscall 行为底层含义和意图的规范。</p>
</div>

<!-- slide data-auto-animate -->

<div class="box-centered">
  <p class="title">• KernelGPT</p>
  <p class="text">下图展示了 KernelGPT 的概述工作流程</p>
  <p class="text">它分为两个阶段进行操作并迭代这个过程进行分析</p>
  <img class="img-fit" src="./assets/Figure_4.png" />
</div>

<!-- slide vertical data-auto-animate -->

<div class="box-centered">
  <p class="title">• KernelGPT</p>
  <p class="text">这种管道结构使得 LLM 能够专注于特定片段，避免来自不相关片段的误导</p>
  <img class="img-fit" src="./assets/Figure_4.png" />
</div>
