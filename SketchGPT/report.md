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

**SketchGPT: A Sketch-based Multimodal Interface for Application-Agnostic LLM Interaction**
UIST '25

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">背景与挑战</p>
  <li class="text list">当前交互局限：主要依赖文本或简单图形界面</li>
  <li class="text list">核心痛点：文本提示（Prompting）难以精确描述空间、形状或图形细节</li>
  <li class="text list">用户负担：需要花费大量精力描述上下文和意图</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">解决方案：SketchGPT</p>
  <li class="text list">新型交互范式：直接在系统界面上同时使用 <span style="color: orange">手绘 (Sketch)</span> + <span style="color: cyan">语音 (Speech)</span></li>
  <li class="text list">Application-Agnostic (与应用无关)：</li>
  <li class="text list list-2">作为一个透明覆盖层运行在操作系统之上</li>
  <li class="text list list-2">不依赖特定软件插件，通过“看”屏幕和模拟操作来控制任意软件</li>
</div>

<!-- slide -->

**形成性研究 (Formative Study)**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">Wizard-of-Oz 实验</p>
  <li class="text list">目的：探究用户在无技术限制下的自然交互偏好</li>
  <li class="text list">设置：10名参与者，幕后由真人（巫师）模拟超级AI</li>
  <li class="text list">发现：用户倾向于同时（Concurrent）使用两种模态，且二者语义互补</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">模态分工 (Content)</p>
  <p class="text font-larger">草图 (Sketch) 的作用：</p>
  <li class="text list">指代 (Reference)：圈出屏幕上的元素</li>
  <li class="text list">空间定位 (Spatial)：画箭头表示移动位置</li>
  <li class="text list">图形描绘 (Graphic)：画草图生成图片</li>
</div>

<!-- slide vertical -->

<div class="box-centered" data-auto-animate>
  <p class="title">模态分工 (Content)</p>
  <p class="text font-larger">语音 (Speech) 的作用：</p>
  <li class="text list">指令 (Command)：发出动作动词（如“计算”、“移动”）</li>
  <li class="text list">细节补充 (Detail)：指定颜色、具体的字词内容</li>
</div>

<!-- slide -->

**系统架构与实现**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">多智能体 (Multi-Agent) 框架</p>
  <li class="text list">核心技术：Chain-of-Thought (思维链 CoT)</li>
  <li class="text list">输入数据：</li>
  <li class="text list list-2">交互上下文 (屏幕截图)</li>
  <li class="text list list-2">草图笔画 (点集)</li>
  <li class="text list list-2">语音话语 (转录文本)</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">意图解读四步走</p>
  <li class="text list">1. 通用意图代理 (General Intention)：生成自然语言的大意描述作为指导</li>
  <li class="text list">2. 草图分割代理 (Sketch Segmentation)：使用 DBSCAN 聚类 + AI 修正语义</li>
  <li class="text list">3. 话语重组代理 (Utterance Reorganization)：清理语音废话，重组语义</li>
  <li class="text list">4. 意图对齐代理 (Intention Alignment)：将笔画与语音语义进行时空对齐</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">反馈与执行</p>
  <li class="text list">工具感知反馈 (Tool-aware Feedback)：</li>
  <li class="text list list-2">将意图转化为 PyAutoGUI 操作、图像API调用等</li>
  <li class="text list">用户确认 (Human-in-the-loop)：</li>
  <li class="text list list-2">提供任务列表和预览框</li>
  <li class="text list list-2">用户可接受、拒绝或要求重生成</li>
</div>

<!-- slide -->

**评估与结论**

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">评估研究 (Evaluation)</p>
  <li class="text list">对比实验 (SketchGPT vs. 单模态)：</li>
  <li class="text list list-2">任务负荷 (NASA-TLX) 显著降低</li>
  <li class="text list list-2">交互效率更高 (无需手写文字，无需口述复杂坐标)</li>
  <li class="text list">典型场景探索 (Exploration)：</li>
  <li class="text list list-2">在 Word, Excel, PPT 中应用</li>
  <li class="text list list-2">用户满意度高 (>5/7分)，认为交互新颖自然</li>
</div>

<!-- slide vertical -->

<div class="box-centered">
  <p class="title">核心结论</p>
  <li class="text list">"A Sketch Worth a Thousand Words"</li>
  <li class="text list list-2">草图极大地降低了空间布局和形状描述的难度</li>
  <li class="text list">"Several Words Simplify a Detailed Sketch"</li>
  <li class="text list list-2">语音简化了复杂绘图和手势记忆的负担</li>
  <li class="text list">SketchGPT 证明了多模态系统级交互是提升 LLM 交互效率的有效路径</li>
</div>
