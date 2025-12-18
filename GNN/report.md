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
  - Graph Neural Network
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

## **Graph Neural Network (GNN)**

Standard Neural Networks (like CNN/RNN) require **Grid** or **Sequence** data.
Graphs differ: **No fixed order**, **Variable neighbors**.

**The Solution: Message Passing**
Each node becomes a "collector", gathering info from friends.

---

## **Graph Neural Network (GNN)**

![](assets/GNN-gathering.png)

---

#### **Message Aggregation**

For each **Node** $v$, we aggregate the information from its neighbors $N(v)$ and store the aggregated message $m$:

The calculation is done layer by layer.

- For the $l$-th layer:

$$m_i^{(l + 1)} = \epsilon(\{h_i^{(l)}, h_j^{(l)}, e_{ij}: v_j\in N(v_i)\})$$

where $\epsilon$ is the aggregation function, $h_i, h_j$ is the hidden state of node $v_i, v_j$, $e_{ij}$ is the state between node $v_i$ and $v_j$

---

#### **Status Update**

We update the hidden state of node $v$ with the aggregated message $m_i$ and old hidden state $h_i^{(l)}$:

$$h_i^{(l + 1)} = \sigma(h_i^{(l)}\textcircled{+}m_i)$$

where $\sigma$ is the activation function, and $\textcircled{+}$ is combination method.

---

#### **Usage**

- **Social Network Analysis**: User Classification, Community Detection, Information Propagation
- **Recommendation System**: Item Recommendation, User Preference Prediction
- **NLP**: Relation Extraction
- etc.

---

#### **Why do we Attack Graphs?**

It is not just about being malicious; it's about **Safety**.

-   **Financial Fraud**: Fraudsters disguise themselves by connecting to normal users.
-   **Social Botnet**: Bots follow real celebrities to look "real".
-   **Robustness Testing**: Finding bugs before the bad guys do.

---

<!-- header: Background: Adversal Attack -->

#### **Adversarial Attack**

Let's use an attack on **image** as an example:

By adding small changes to the image, the prediction result will be completely different.

![](assets/image-attack.png)

---

### Attack on **Social Network**

![](assets/attack.png)

---

#### **Attack on Graph Data**

Unlike images where we change pixels, in graphs, we change the **Structure**.

- **Perturbation**: Adding or deleting edges (relationships).
- **Goal**: Make the GNN classify a specific node (or many nodes) incorrectly.

**Example**:
> *Attacker adds a few fake "friend" connections to a user, causing the system to classify a normal user as a "bot".*

---

## **The Reality Gap**

Most existing research focuses on **Tiny Datasets** (e.g., Cora, PubMed ~20k nodes). 

However, real-world graphs are **Massive**:

![](assets/existing-works.png)

---

## **Why is Scale a Problem?**

**Memory Explosion!** 💥

To attack a graph optimally, we often need to calculate gradients for **all possible edges**.

- **Space Complexity**: $O(n^2)$ (Quadratic)
- For a graph with millions of nodes, storing the dense adjacency matrix requires **Exabytes** of memory. 

---

<!--
_class: title-page
header: Robustness of GNNs at Scale
-->

## Robustness of GNNs at Scale
#### **NeurIPS 2021**

---

### **Challenge 1: The "Wrong" Goal**

Attackers usually use **Cross Entropy (CE)** loss to guide the attack.

On large graphs with small budgets, CE wastes energy attacking nodes that are **already wrong** or **too hard** to flip. 

![](assets/budgets.png)

---

### **Deep Dive: The Problem with Cross Entropy (CE)**

To attack a model, we usually maximize the **Cross Entropy Loss**.
$$\mathcal{L}_{CE} = - \log(p_{correct\_class})$$

-   It wants the probability of the correct class ($p_{correct}$) to be **0**.
-   Even if the node is already misclassified (e.g., $p_{correct} = 0.4$, predicted class is wrong), CE is **not satisfied**. It pushes $p_{correct}$ towards $0.0001$.

> CE has an "Infinite Appetite". It keeps attacking nodes that are **already dead** (misclassified).

---

### **Why CE Fails at Scale (Global Attack)**

Imagine you have **1 Million Nodes** but budget to flip only **1,000 Edges**.

**The CE Trap:**
1.  **Gradients**: CE generates huge gradients for nodes that are "very wrong" (low confidence).
2.  **Budget Waste**: The attack spends the limited budget making "wrong" nodes "more wrong" (e.g., probability $0.4 \rightarrow 0.01$).
3.  **Outcome**: The **Accuracy** (0/1 count) doesn't change! You wasted ammo on dead targets.

> We need a loss function that says: **"This node is dead, move on to the next one!"**

---

### **Solution: Defining the "Margin"**

First, let's look at the **Logits** (raw output $z$) instead of Probabilities.

We define the **Classification Margin** $\psi$:
$$\psi = z_{correct} - \max_{c \neq correct} z_{c}$$

-   **$\psi > 0$**: Node is **Correct** (Safe).
-   **$\psi < 0$**: Node is **Wrong** (Misclassified).
-   **$\psi \approx 0$**: Node is on the **Decision Boundary** (Fence-sitter).

*Our Goal: Only attack when $\psi > 0$ but close to 0.*

---

### **The "Magic" of Tanh Margin**

The paper proposes a new loss: **Tanh Margin**.
$$\mathcal{L}_{Tanh} = \tanh(\psi) = \tanh(z_{correct} - z_{best\_other})$$

**How the Gradients Work (The Math of Tanh):**
The derivative of $\tanh(x)$ is $1 - \tanh^2(x)$. This is a **Bell Curve**!

-   **If $\psi \ll 0$ (Already Wrong):** Gradient $\approx 0$. **(Stop Attacking)** 
-   **If $\psi \gg 0$ (Too Safe):** Gradient $\approx 0$. **(Ignore)**
-   **If $\psi \approx 0$ (Boundary):** Gradient is **MAX**. **(Attack Here!)**

*Result: The attack strictly targets the most vulnerable nodes.*

---

### **Challenge 2: Memory Limits**

Traditional attacks (like PGD) try to optimize the **entire** adjacency matrix at once.

$$\Theta(n^2) \text{ Memory Usage}$$

For 1 million nodes, $n^2$ is $1,000,000,000,000$ parameters. **Impossible to store.** 

---

### **Solution: PR-BCD**

**Projected Randomized Block Coordinate Descent** 

Instead of looking at the whole graph, we look at a small random **Block** at a time.

![](assets/PR-BCD.png)

- **Memory**: $O(b)$ instead of $O(n^2)$.
- **Speed**: We only update a subset of edge probabilities per step.

---

### **How PR-BCD Actually Works (Step-by-Step)**

We treat the adjacency matrix $A$ as a set of continuous probabilities $P_{ij} \in [0,1]$.

**The Loop:**
1.  **Sample**: Randomly pick a small block of $b$ edges (e.g., $10^5$ out of $10^{12}$).
2.  **Gradient**: Calculate gradients **only** for these $b$ edges.
3.  **Update**: Adjust the probability of flipping these edges.
4.  **Project**: Ensure the sum of probabilities fits our Budget $\Delta$.

---

### **The "Survival of the Fittest" Heuristic**

How do we find the best edges if we don't look at all of them?

**Resampling Strategy**:
At the end of every epoch, we look at our block of $b$ edges:
- **Keep**: The edges with high flip probabilities (The "Strong" attackers).
- **Discard**: The edges with low probabilities (The "Weak" ones).
- **Refill**: Randomly sample new edges from the graph to replace the discarded ones.

*Result: Over time, the block accumulates the most vulnerable edges in the graph.*

---

### **Challenge 3: Defending the Graph**

GNNs aggregate messages from neighbors.
**Sum** or **Mean** aggregation is vulnerable: one malicious "extreme" neighbor ruins the average.

$$\text{Mean}([1, 2, 2, 100]) = 26.25 \quad (\text{Skewed!})$$

$$\text{Median}([1, 2, 2, 100]) = 2 \quad (\text{Robust!})$$

---

### **Solution: Soft Median**

The paper introduces **Soft Median** aggregation. 

- **Robust**: It ignores outliers (attacked edges).
- **Differentiable**: Unlike standard median, it can be trained via backpropagation using a "Temperature" parameter.
- **Scalable**: Much faster and less memory-intensive than previous robust methods (like Soft Medoid).

---

### **Implementing Soft Median**

Standard Median is **sorting**, which breaks backpropagation (gradients can't flow through "sort").

**Soft Median** approximates it using a **Weighted Average**:

$$\mu_{\text{Soft}} = \sum_{x \in \text{Neighbors}} w_x \cdot x$$

The "magic" is in how we calculate the weights $w_x$.

---

### **The Weight Calculation**

We want **Outliers** (attacked edges) to have **Zero Weight**.

1.  Calculate **Distance** $d_i$ between node $i$ and the dimension-wise median.
2.  Compute Weight using **Softmax**:

$$w_i = \text{Softmax}\left( - \frac{d_i}{T} \right)$$

- If distance $d_i$ is **High** (Outlier) $\rightarrow w_i \approx 0$ (Ignored).
- If distance $d_i$ is **Low** (Normal) $\rightarrow w_i$ is high.

---

The temperature parameter $T$ controls the "Softness".

![](assets/soft-median.png)

By tuning $T$, we can ignore adversarial attacks.

---

### **Experimental Results**

The authors tested on **Papers100M** (111 million nodes). 

**Attack Performance**:
- PR-BCD is highly effective even with <11GB memory. 

**Defense Performance**:
- Under attack, standard GNN accuracy drops to ~1%.
- **Soft Median** defense keeps accuracy high (~30-80% depending on setup). 

---

<!--
_class: title-page
header: SGA: Simplified Gradient-based Attack
-->
## SGA: Simplified Gradient-based Attack
#### **The Art of "Less is More"**
(TKDE 2021)

---

### **The Bottleneck of Global Attacks**

Standard gradient attacks (like PGD) need to calculate gradients for **every possible edge**.

-   **Graph Size**: $N$ nodes.
-   **Possible Edges**: $N^2$ (Dense Matrix).
-   **For Reddit Dataset**: $230,000$ nodes $\rightarrow$ $53$ Billion possible edges.

**Result**: GPU Memory Overflow (OOM) 💥.

> **SGA's Question**: Do we really need the *entire* graph to attack *one* node?

---

### **Step 1: Subgraph Extraction**

**Intuition**: The "Butterfly Effect" in GNNs is limited.
A change 10 hops away usually won't affect the target node significantly.

**SGA Strategy**:
Extract a **$k$-hop subgraph** centered at the target node (usually $k=2$).

-   **Reduced Space**: From $N$ nodes to just local neighbors (~100s).
-   **Efficiency**: Computations become instantaneous.

---

### **Step 2: Enlarging the Search Space**

Just deleting existing edges is not enough. We want to **Add** malicious edges.
But connecting to *whom*?

---

**Potential Nodes**:
SGA identifies nodes outside the subgraph that carry strong "wrong class" signals.

![](assets/SGA-attack.png)

We add these **Potential Nodes** to the subgraph to calculate their gradients.

---

### **Step 3: The "Vanishing Gradient" Problem**

We use a simplified model (SGC) to get gradients: $\nabla = \frac{\partial \mathcal{L}}{\partial A}$.

**The Trap**:
If the model is **Too Confident** (e.g., Probability = 0.9999), the loss curve is flat.
$$\text{Flat Curve} \rightarrow \text{Gradient} \approx 0$$

The attacker gets **No Signal** on how to improve the attack.

---

### **Step 4: Gradient Calibration**

**Solution**: Introduce a Scale Factor $\epsilon$ (Temperature).

$$P = \text{Softmax}\left( \frac{\text{Logits}}{\epsilon} \right)$$

By dividing logits by $\epsilon$ (e.g., $\epsilon=5.0$):
1.  The probability distribution becomes **Softer**.
2.  The gradients become **Non-Zero**.
3.  The attacker "sees" the direction again.

---

### **Step 5: The Iterative Attack Loop**

SGA attacks sequentially to be precise.

![](assets/attack-loop.png)

> **Note**: If we add an edge to a distant node, we must **Update the Subgraph** to include that node's neighbors for the next step.

---

<!--
_class: title-page
header: GRB: Graph Robustness Benchmark
-->

## GRB: Graph Robustness Benchmark
#### **NeurIPS 2021**

---

### **The "Wild West" of GNN Security**

Before this paper, comparing different defense methods was like comparing **Apples to Oranges**.

- **Different Datasets**: One paper uses *Cora*, another uses *Reddit*.
- **Different Settings**: One assumes the attacker knows everything (White-box), another assumes nothing (Black-box).
- **Tiny Scales**: Most defenses only worked on graphs with < 5,000 nodes.

> **Problem**: We don't know which defense is actually robust in the real world. 

---

### **The 3 Major Flaws in Previous Benchmarks**

1.  **Unrealistic Assumptions**:
    Attackers often assume they can modify *any* edge. In reality, you can't just delete a Facebook friendship between two strangers. 
2.  **Lack of Unity**:
    No standardized protocol. Paper A attacks 5% of nodes; Paper B attacks 10%. Results are incomparable. 
3.  **Scalability Issues**:
    Existing benchmarks ignored large graphs. A defense that works on 1,000 nodes might crash on 1 million. 

---

#### **GRB: The Unified Framework**

GRB proposes a standardized pipeline to fix these issues.

![](assets/GRB-framework.png)

It focuses on **Reproducibility** and **Realism**. 

---

### **Scenario 1: Modification (The Classic View)**

Attackers modify the **Existing Structure** of the graph.

-   **Action**: Add/Remove edges between existing nodes.
-   **Perturb Features**: Change the attributes of a user.
-   **Reality Check**: Hard to do in real life. (Requires hacking the database or compromising specific users). 

---

### **Scenario 2: Graph Injection (Realism)**

**Modification Attack** (Modifying existing edges) is hard in reality.
> *You cannot force two strangers on Facebook to become friends.*

**Injection Attack** (Adding new nodes) is realistic.
> *You CAN create 100 fake bot accounts and have them follow the target.*

GRB focuses heavily on this **Injection** scenario.

---

## **Summary**

| Paper | Problem | Key Innovation | Takeaway |
| :--- | :--- | :--- | :--- |
| **Robustness at Scale** | Memory $O(N^2)$ | **PR-BCD**: Random Block Sampling<br>**Tanh Margin**: Better Loss | Optimize sparse blocks; Stop attacking dead nodes. |
| **SGA** | Time Complexity | **Subgraph**: Ignore far nodes<br>**Calibration**: Fix vanishing grad | Local info is enough; Scale factor matters. |
| **GRB** | Evaluation Chaos | **Standardization**: Unified Pipeline<br>**Leaderboard**: Matrix Evaluation | Focus on **Injection** attacks and **Scalable** datasets. |

---

### **Visualizing Injection Attack**

![](assets/GRB-attack.png)

---

### **The Unified Rules**

To make comparisons fair, GRB sets strict rules:

1.  **Black-box**: Attackers know the graph data, but **NOT** the model weights or defense strategy. 
2.  **Inductive**: The model is trained on one set of nodes, but attacked on **Unseen Nodes** (New users). 
3.  **Evasion**: The attack happens during **Inference** (Deployment), not during training. 

---

### **Data: From Tiny to Huge**

GRB includes 5 datasets to test **Scalability**.

| Dataset | Type | Nodes | Edges | Scale |
| :--- | :--- | :--- | :--- | :--- |
| **grb-cora** | Citation | ~2.6k | ~5k | Tiny |
| **grb-flickr** | Social | ~89k | ~450k | Medium |
| **grb-reddit** | Social | ~232k | ~11.6M | **Large** |
| **grb-aminer** | Academic | ~659k | ~2.8M | **Large** |



---

### **Not All Nodes are Equal**

GRB discovered that **Degree** (number of connections) determines robustness.

-   **Low Degree**: Easy to attack. (Few friends = easy to influence).
-   **High Degree**: Hard to attack. (Many friends = stable opinion).

**The Splitting Scheme**:
GRB splits the test set into **Easy, Medium, and Hard** subsets based on node degree to analyze defenses in depth. 

---

### **The "Arms Race" Leaderboard**

A defense isn't good if it only stops *one* specific attack.
GRB evaluates a **Matrix** of battles.

This prevents "overfitting" a defense to a specific attack method. 

---

$$\text{Score} = \text{Weighted Average against ALL Attacks}$$

![](assets/GRB-defense.png)
