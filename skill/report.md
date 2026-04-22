---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #FFFFFF
color: #303133
footer: Skills for agents | Chen Qi @ 2025
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
_header: Skills for agents
-->

## **Skills for agents**
Recent Papers

---

### **Skill?!**

*一个没有技能的 Agent 该怎么完成任务？*
- 从头开始思考每一步并顺序执行，例如要安装思题目学习平台：
  - 打开浏览器
  - 输入网址
  - 点击下载按钮
  - 运行安装程序
  - 完成安装

如果要安装别的东西，还是要重复思考一遍相同的内容

---

### **Skill?!**

🤓👆我们可以把这个步骤打包成一个“技能”
- 把上面的步骤打包成“安装软件()”技能
- 下次需要安装软件，只需要调用这个技能就行，不需要重新想步骤

**好处？**

- 不用一步一步思考，更加快，更节省 token
- 来自成功经验总结，不容易出错，并且可以保存
- 来自强力模型总结的技能可以给小型模型用

---

<!--
_class: title-page
header: PolySkill: Learning Generalizable Skills Through Polymorphic Abstraction for Continual Learning
-->

## **PolySkill**
### Skills through polymorphic abstraction
ICLR 2026

---

### **什么是网页智能体？**

- 大模型驱动的 AI 程序
- 能像人一样操作网页
- 能够完成用户指定的任务

### **现有网页智能体的核心问题**

- 在一个网站上学会的技能，没有办法应用到另一个网站上
- 学会新网站技能后，原有网站性能显著下降

---

### **使用软件工程的“多态”**

- 先制定一个抽象技能，定义“做什么”
- 再为每个不同的情况制定特定的方法，定义“怎么做”

**例如：**
- 每个购物网站都需要“搜索商品”
- 为每个网站实现单独的搜索方法

---

![](./assets/polyskill.png)

---

### **遇到需要新学习的知识？**

- 直接可以根据抽象模板学习
- 学习速度可以大幅加快
- 任务成功后，在模板中记录新的实现
- 技能库中记录了所有学习到的技能，越用越厉害

![](./assets/polyskill-learning.png)

---

<!-- 
_class: title-page
header: AutoSkill: Experience-driven Lifelong Learning Via Skill Self-evolution
-->

## **AutoSkill**
### Experience-driven Self-evolution

---

**在实际应用中，用户会反复表达稳定的偏好和要求：**

- 减少幻觉
- 遵循指定写作规范
- 避免过于技术化的措辞
- 遵守固定的工作流程

但这些交互经验**很少被转化为可复用的知识**，导致用户每次开启新会话都需要重新说明自己的要求。

已有的解决方案要么成本过高，要么无法长期维护

---

### **AutoSkill**

AutoSkill 是一个**无需训练基础模型**的经验驱动终身学习框架：

> 不把交互经验仅仅当作记忆，而是当作技能形成的来源。从用户交互中抽象出可复用的行为，将其固化为**显式、可编辑、带版本控制的 SKILL.md 技能工件**，并在未来的请求中动态注入相关技能，实现能力的持续积累。

它是一个模型无关的插件层，可以兼容所有现有 LLM，并且支持技能在不同代理、用户和任务之间共享和迁移。

---

### 解决方案：

#### **左循环：用技能回答问题**

    收到用户请求 → 找到相关技能 → 用技能生成更好的回答

#### **右循环：从对话中学技能**

    对话结束后 → 提取新的技能 → 更新技能库

#### 所有操作**都不修改 AI 模型的参数**

---

![](./assets/circle.png)

---

<!--
_class: title-page
header: XSkill: Continual Learning from Experience and Skills in Multimodal Agents
-->

## **XSkill**
### Continual Learning from Experience and Skills

---
