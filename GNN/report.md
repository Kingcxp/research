---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #FFFFFF
color: #303133
footer: GNN: Adversal Attack and Defence | Chen Qi @ 2025
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

<!--
header: GNN: Adversal Attack and Defence
_class: title-page
-->

## GNN: 
## **Adversal Attack and Defence**

---

<!-- _header: GNN: Catalog -->

## Catalog
- **Background**
  - Graph
  - Graph Neuro Network
  - Adversal Attack
- **Why Large Scale Graph**
- **How to**
  - Related Work

---

<!-- header: Background: Graph -->

## **Graph**
Graph is a kind of data structure, which is composed of **Nodes** and **Edges**.
**Nodes** are connected by **Edges** to express the relationship.

![](assets/Undirected.png)

---

## **Graph**
Graph is a kind of data structure, which is composed of **Nodes** and **Edges**.
The **Edges** can either be directed or undirected.
![](assets/Directed.png)

---

<!-- header: Background: Graph Neural Network -->

## **Graph Neural Network**

![](assets/fully-connected-neuro-network.png)

**Neuro Network** can convert some vector into another vector.

But it requires the data to be **Sequential**

---

## **Graph Neural Network**

If we need to use neural network on a graph, we have to convert the graph into a **Sequence**.

This causes the loss of information:
- **Nodes** are in order, but the order does not matter in graph.
- **Edges** are not considered.
- We cannot percept the **Graph Structure**.

So we need to find a better way to convert **Nodes** into **Vectors**.

---

#### **Message Aggregation**

For each **Node** $v$, we aggregate the information from its neighbors $N(v)$ and store the aggregated message $m$:

The commonly used aggregation functions are:
- **Sum**: $\sum_{v_j \in N(v_i)} RELU(W\cdot h_j + b)$
- **Mean**: $\frac{1}{|N(v_i)|} \sum_{v_j\in N(v_i)}h_j$
- **Max Pooling**: $m_i = \max_{v_j\in N(v_i)}\{h_j\}$

where $h_j$ is the hidden state of node $v_j$, $W$ and $b$ are learnable parameters.

---

#### **Status Update**

We update the hidden state of node $v$ with the aggregated message $m_i$ and old hidden state $h_i^{(l)}$:

$$h_i^{(l + 1)} = \sigma(h_i^{(l)}\textcircled{+}m_i)$$

where $\sigma$ is the activation function, and $\textcircled{+}$ is concatenation or linear transformation.
