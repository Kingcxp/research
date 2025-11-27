---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #141414
color: #E5EAF3
footer: Chen Qi @ 2025
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
    background-color: #1D1E1F;
    height: 64px;
    width: 100%;
    top: 0;
    left: 0;
  }

  section::after {
    content: "";
    background-color: #1D1E1F;
    margin-top: auto;
    height: 64px;
    width: 100%;

    display: flex;
    flex-direction: row-reverse;
    z-index: 0;
  }

  header {
    z-index: 1;
    color: #E5EAF3;
  }

  footer {
    z-index: 1;
    color: #E5EAF3;
  }

  img {
    max-height: 50vh;
    max-width: 100%;
  }

  strong {
    color: #409EFF !important;
  }
---

<!-- _class: title-page -->

## Overview of LLM:
## **Reasoning** and **Optimization**

---

<!-- _header: Catalog -->

- #### Transformer
  - What does NLP want to solve
  - Before the Transformers
  - Transformer
- #### LLM
  - The challenge
  - KV Cache
- #### Optimizations
---

<!--
_class: title-page
_header: Transformer
-->

## NLP:
## **What We Want to Solve**
## And **How to Solve It**

---

<!-- header: NLP (Nature Language Processing) -->

- ###### What does NLP want to solve?

---

- ###### What does NLP want to solve?

The application for NLP is **translation**.

In translation, we want to translate a sentence to another sentence.

---

- ###### What does NLP want to solve?

The application for NLP is **translation**.

In **translation**, we want to translate a sentence to another sentence.

The sentence and the translation result can be **anything**, for example:

![](assets/translate-example.png)

---

- ###### What does NLP want to solve?


![](assets/translate-example.png)

To put it more vividly, given the sentence as context, we need to **predict what's next**.

---

- ###### How to represent words so that computers can **understand** them?

---

- ###### How to represent words so that computers can **understand** them?
  
- **Solution**: Represent words as **vectors** (embeddings)

![](assets/embedding.png)

- Through the help of a pretrained matrix, each token is converted into a vector of size $d_{model}$, which every number in the vector represents a semantic feature of the token.

---

<!-- header: Before the **Transformers** -->

- ###### So before the **Transformers**, how did we translate the embeddings?

---

- ###### So before the **Transformers**, how did we translate the embeddings?

**RNN(Recurrent Neural Network)** is commonly used for translation

![](assets/fully-connected-neuro-network.png)

**Neuro Network** can convert some vector into another vector.

---

- ###### How does **RNN** work?

Based on the **Fully Connected Neural Network**, **RNN** can remember the weighed values for all the hidden layers of the previous embedding.

This helps **RNN** to take in embeddings one by one and remember the context.

![](assets/rnn-translate.png)

---

- ###### How does **RNN** work?

![](assets/rnn-translate.png)

After taking in all the embeddings, **RNN** can remember the combined context of all the tokens.

Afterwards, **RNN** can output a vector representing the chance of which token should be the next token.

Similarly, **RNN** takes in the token and predicts the next token.

---

<!-- header: Drawbacks of **RNN** -->

- Then why **RNN** should be replaced?

**RNN** can only take in the embeddings one by one, which means that **RNN** cannot take in a token before all the tokens before it are taken in.

![](assets/rnn-translate.png)

This takes really a long time, and is hard to be optimized.

Also, as **RNN** read more tokens, the context may be lost.

---

<!--
_class: title-page
header: Transformer
-->

## Transformer:
## **A Brand New Approach**

###### Attention is All You Need (NeuroIPS 2017)

---

- ###### How Transformer improves upon RNN?

**RNN** processes data sequentially (Time-step $t$ relies on $t-1$).
**Transformer** processes data **in Parallel**.

1.  **Parallelism**: It takes the whole sentence at once. No more waiting for the previous word.
2.  **Long-term Dependency**: Through **Self-Attention**, the first word can directly "see" the last word, regardless of distance.

---

<!-- _header: Transformer - **The Structure** -->

- ###### The Big Picture: Encoder-Decoder Structure

Transformer follows the standard **Encoder-Decoder** structure.

![](assets/transformer-overview.png)

---

<!-- _header: Transformer - **Encoder** -->

- ###### Zooming into the **Encoder**

The Encoder is a stack of $N$ identical layers (e.g., $N=6$).
Each layer consists of two main sub-layers:

1.  **Multi-Head Self-Attention**: To find relationships between tokens.
2.  **Feed-Forward Network (FFN)**: To process and digest the information.

![](assets/transformer-encoder-layer.png)

---

<!-- header: Transformer - **Multi-Head Self-Attention** -->

![](assets/embedding.png)
![](assets/qkv.png)

---

- ###### The Calculation of **Q, K, V**
  - **Query ($Q$)**: What I am looking for.
  - **Key ($K$)**: The label/tag of the file.
  - **Value ($V$)**: The actual content of the file.

- **The Attention Score**:
$$Attention(Q, K, V) = Softmax(\frac{Q\times K^T}{\sqrt{d_k}})\times V$$

---

- ###### The Calculation of **Attention**
  - **$Q\times K^T$**: Find the tokens that are related to the query.
  - **$Softmax$ & $\sqrt{d_k}$**: Normalize the scores.
  - **$\times V$**: Extract the meanings of the matched tokens.

- **The Attention Score**:
$$Attention(Q, K, V) = Softmax(\frac{Q\times K^T}{\sqrt{d_k}})\times V$$

---

- ###### Why **Multi-Head**?

Language is complex.

---

- ###### Why **Multi-Head**?

Language is complex.

We split $Q, K, V$ into `n_heads` heads, calculate relationships independently, and then concatenate the results.

![](assets/multi-heads.png)

---

<!-- header: Transformer - **Feed Forward Network & Add & LayerNorm** -->

Attention is for **"Gathering Info from other tokens"**, **FFN** is for **"Processing Info independently"**.

- **Feed Forward Network**: 
  - A simple MLP to digest the context information.
  - Expands the dimension (e.g., $512 \to 2048$) and compresses it back.

- **Add & Norm**: 
  - **Add (Residual)**: $x + Layer(x)$. Prevents information loss.
  - **Norm (LayerNorm)**: Stabilizes the numbers for training.

---

<!-- header: Transformer - **Decoder** -->

- ###### While the **Encoder** understands the past (Context), the **Decoder** predicts the future (Generation).

- **The Goal**: Autoregressive Generation.
Given tokens $x_1, ..., x_{t-1}$, predict $x_t$.

![](assets/decoder-layer.png)

![](assets/decoder.png)

---

- ###### Zooming into the **Decoder**: The generation process

Its layer is similar to the **Encoder**, but a little different:

![](assets/translate-loop.png)

---

<!-- header: Transformer: **Multi-Head Self-Attention (for Decoder)** -->

Suppose we have generated "I", "Love". Now the input is "AI".
We need to calculate the **Query**, **Key**, and **Value** for this **new token** only.

$$
q_{new} = x_{AI} \times W_Q, \quad k_{new} = x_{AI} \times W_K, \quad v_{new} = x_{AI} \times W_V
$$

- **$q_{new}$**: The new query vector. "What relates to 'AI'?"
- **$k_{new}$**: The new key vector. The label for "AI".
- **$v_{new}$**: The new content vector. The meaning of "AI".

---

To predict the next word, "AI" must "look back" at "I" and "Love".
We combine the **Past Keys** with the **New Key**.

The **Values** are the same for all tokens.

$$
K_{total} = Concat([K_{past}, k_{new}])
$$

$$
V_{total} = Concat([V_{past}, v_{new}])
$$

---

We calculate how much the **New Query** matches **All Keys** (Past + Present).

$$
Scores = q_{new} \times K_{total}^T
$$

Finally, we calculate the **Output**.

$$
Output = Softmax(\frac{Scores}{\sqrt{d_k}}) \times V_{total}
$$

- This vector now contains the **Full Context** needed to predict the next word.

---

After all the process, the output vector is projected to the vocabulary size to get probabilities.

$$
Probabilities = Softmax(Output \times W_{vocab})
$$

![](assets/probabilities.png) 

---

<!-- header: Transformer: **KV Cache** -->

- ###### The Naive Inference Strategy (Without Cache)

Let's look at how a standard Transformer generates text **without any optimization**.
To generate the token at step $t$, we must input the **entire sequence** $x_1, ..., x_{t-1}$.

1. **Step 1**: Input `["I"]` $\to$ Compute Attention for 1 token.
2. **Step 2**: Input `["I", "Love"]` $\to$ Compute Attention for 2 tokens.
3. **Step 3**: Input `["I", "Love", "AI"]` $\to$ Compute Attention for 3 tokens.

---

**Do you see the problem?**
We calculated the embedding and attention for "I" in Step 1.
We calculated it **AGAIN** in Step 2.
We calculated it **YET AGAIN** in Step 3.

**As sequence length grows, the "Redundant Calculation" forms a huge pyramid.**

---

Let's do the math for generating a sequence of length $N$.

At **Step $t$** (current length $t$):

---

Let's do the math for generating a sequence of length $N$.

At **Step $t$** (current length $t$):
* We input a matrix of shape $(t \times d)$.

---

Let's do the math for generating a sequence of length $N$.

At **Step $t$** (current length $t$):
* We input a matrix of shape $(t \times d)$.
* **Self-Attention Cost**: $Q(t \times d) \times K^T(d \times t) \to \text{Score}(t \times t)$.

---

Let's do the math for generating a sequence of length $N$.

At **Step $t$** (current length $t$):
* We input a matrix of shape $(t \times d)$.
* **Self-Attention Cost**: $Q(t \times d) \times K^T(d \times t) \to \text{Score}(t \times t)$.
* Matrix Multiplication Complexity: **$O(t^2 \cdot d)$**.

---

Let's do the math for generating a sequence of length $N$.

At **Step $t$** (current length $t$):
* We input a matrix of shape $(t \times d)$.
* **Self-Attention Cost**: $Q(t \times d) \times K^T(d \times t) \to \text{Score}(t \times t)$.
* Matrix Multiplication Complexity: **$O(t^2 \cdot d)$**.

**Total Inference Cost** (Sum of all steps):
$$
\text{Total} = \sum_{t=1}^{N} O(t^2 \cdot d) \approx O(N^3 \cdot d)
$$

---

- ###### The Grand Summary: Why we need optimization?

We face two main enemies in the standard Transformer:

| Problem | Symptom | Source | Solution |
| :--- | :--- | :--- | :--- |
| **Redundant Compute** | Slow Speed ($O(N^3)$) | Re-calculating $K, V$ for past tokens every step | **KV Cache** |

---

<!--
header: Complexity Analysis: **Step-by-Step**
_class: title-page
-->

## The Real Cost:
## **Time & Space** Breakdown

---

<!--
header: Complexity Analysis: **Encoder Layer**
-->

- **QKV**:
  - **Time**: $[n, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(nd_{model}^2)$
- **Self-Attention**: 
  - **Time**: $[n, d_{heads}]\times [d_{heads}, n] \times [n, d_{heads}] \Rightarrow O(n^2d_{heads})$
- **FFN**:
  - **Time**: $[n, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(nd_{model}^2)$

- **Total**:
  - **Time**: $O(nd^2 + n^2d)$

---

<!--
header: Complexity Analysis: **Decoder Layer**
-->

- **QKV**:
  - **Time**: $[1, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(d_{model}^2)$
- **Self-Attention**: 
  - **Time**: $[1, d_{heads}]\times [d_{heads}, n+t] \times [n+t, d_{heads}] \Rightarrow O(td_{heads})$
- **FFN**:
  - **Time**: $[1, d_{model}]\times [d_{model}, d_{model}] \Rightarrow O(d_{model}^2)$

- **Total(For generating length of m)**:
  - **Time**: $O(m(d^2 + (n+m)d))$

---

<!--
header: Transformer: **Training**
_class: title-page
-->

## Transformer:
## **How to Train**

###### From Randomness to Intelligence

---

- ###### The Transformation

**Training** = Tuning the matrices ($W$) to minimize error.

![](assets/training-overview.png)

---

- ###### Step 1: We construct the **Question** (Input) and **Answer** (Target) from the raw text.

![](assets/train-target.png)

---

- ###### Step 2: The Problem of "Cheating"

If we feed the whole sequence, Position 1 ("I") can see Position 2 ("Love").
This is **Data Leakage**.

![](assets/cheating.png)

---

- ###### Step 3: The Mask Matrix (The Blindfold)

We add $-\infty$ to the upper triangle to block future connections.

$$
\text{Attention Scores} + \text{Mask} \rightarrow \text{Softmax}
$$

$$
\begin{bmatrix} 
\text{Sim}(I, I) & \text{Sim}(I, Love) \\ 
\text{Sim}(Love, I) & \text{Sim}(Love, Love) 
\end{bmatrix}
+
\begin{bmatrix} 
0 & \mathbf{-\infty} \\ 
0 & 0 
\end{bmatrix}
\approx
\begin{bmatrix} 
\text{Score} & \mathbf{0} \\ 
\text{Score} & \text{Score} 
\end{bmatrix}
$$

* **Result**:
  * Row 1 can **ONLY** see Col 1.
  * Row 2 can see Col 1 & 2.

---

- ###### Step 4: Parallel Loss Calculation

We calculate errors for **ALL** positions simultaneously.

![](assets/loss.png)

$$
\text{Total Loss} = \text{Loss}_1 + \text{Loss}_2 + \text{Loss}_3
$$

---

- ###### Step 5: The Learning Cycle

![](assets/learning.png)

**Repeat this loop billions of times.**

---

- ###### Comparison: Training vs Inference

| Feature | Training | Inference |
| :--- | :--- | :--- |
| **Strategy** | **Parallel** (Teacher Forcing) | **Serial** (Autoregressive) |
| **Input** | Whole Sentence | Past Tokens Only |
| **Future** | **Masked** (Artificial) | **Non-existent** |
| **Complexity** | $O(N^2)$ (One Pass) | $O(N^3)$ (Loop) |

---

<!--
header: Transformer: **Memory IO**
_class: title-page
-->

## Transformer:
## **Memory IO**

###### The cost in the hardware reality

---

Understanding **Data Movement** is key to optimization.

![](assets/memory.png)

**Rule**: Computation can ONLY happen in SRAM. 
Data must be loaded **HBM $\to$ SRAM** to be processed, then written back.

---

- IO Complexity of Self-Attention:

| Operation               | Read HBM           | Write HBM   | Complexity            |
| ---------------- | -------------- | ------ | -------------- |
| $S = QK^T$        | Q, K (2Nd)     | S (N²) | O(Nd + N²)     |
| $P = softmax(S)$ | S (N²)         | P (N²) | O(N²)          |
| $O = PV$         | P, V (N² + Nd) | O (Nd) | O(Nd + N²)     |
| **Total**     |                  |                |         **O(Nd + N²)** |


---

- ###### Phase 1: Prefilling (**The Encoder**)

**Scenario**: Input $n$ tokens at once.

* **Operation**: Matrix Multiplication (Input $\times$ Weights).
* **Memory Pattern**:
    1.  Load Weights ($W$) from HBM to SRAM **Once**.
    2.  Compute for **ALL $n$ tokens** in parallel on SRAM.
    3.  Write KV Cache to HBM.

**Status: Compute Bound (Good)**

---

- ###### Phase 1: Prefilling (**The QKV**)

**Target**: Calculate $S = QK^T, P = Softmax(S), O = PV$

1. Read $Q, K$ from HBM $\to$ Compute $S$ $\to$ **Write $S$ to HBM**.
2. Read $S$ from HBM $\to$ Compute $P$ $\to$ **Write $P$ to HBM**.
3. Read $P, V$ from HBM $\to$ Compute $O$ $\to$ **Write $O$ to HBM.**

Frequently moving data between HBM and SRAM is **expensive**.

---

- ###### Phase 2: Decoding (Generation)

**Scenario**: Generate **1 single token**.

* **Operation**: Vector-Matrix Multiplication.
* **Memory Pattern**:
    1.  Load the **Entire Model Weights** ($d\times d$) from HBM.
    2.  Load the **Entire KV Cache** ($N\times d$) from HBM.
    3.  Compute for just **1 token**.

> **Status: Memory Bound (Bad)**
> We moved GBs of data just to calculate a tiny vector.

---

- ###### Arithmetic Intensity

| Phase | Input Size | Math (FLOPs) | Memory Access | Intensity | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Prefill** | $N$ (Large) | $O(N \cdot d^2)$ | $O(d^2)$ (Weights) | **High** | Compute Bound |
| **Decoding** | 1 (Tiny) | $O(d^2)$ | $O(d^2)$ (Weights) | **Low (~1)** | **Memory Bound** |

**Decoding is slow because of Low Arithmetic Intensity.**

---

<!--
header: Transformer: **FlashAttention Optimization**
_class: title-page
-->

## Transformer Optimization:
## **FlashAttention**

###### Fast and Memory-Efficient Exact Attention with IO-Awareness

---

- ###### Phase 1: Prefilling (**The QKV**)

**Target**: Calculate $S = QK^T, P = Softmax(S), O = PV$

1. Read $Q, K$ from HBM $\to$ Compute $S$ $\to$ **Write $S$ to HBM**.
2. Read $S$ from HBM $\to$ Compute $P$ $\to$ **Write $P$ to HBM**.
3. Read $P, V$ from HBM $\to$ Compute $O$ $\to$ Write $O$ to HBM.

Frequently moving data between HBM and SRAM is **expensive**.

**The bottleneck is reading/writing $N^2$ elements to HBM.**

---

- ###### FlashAttention: The Core Idea

**"IO-Awareness"**: minimize HBM accesses.

Two key techniques:
1. **Tiling**: Compute attention by **blocks**. Load small blocks of $Q, K, V$ to SRAM, compute, and update output without writing full matrix $S$ to HBM.
2. **Recomputation**: Do not save the attention matrix for backward pass. Recompute it on-the-fly.

---

<!-- header: FlashAttention: **Tiling** -->

- ###### Technique 1: Tiling (Forward Pass)

Instead of computing the full $N \times N$ matrix, we  try to **split $Q, K, V$ into blocks** that fit in **SRAM**.

![](assets/split.png)

---

<!-- header: Tiling: **How to Calculate?** -->

- ###### The Softmax Problem

Standard Softmax needs the **entire row** to calculate the normalization factor (denominator).

$$Softmax(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{N} e^{x_j}}$$

---

<!-- header: Softmax: **Merging the blocks** -->

![](assets/merge-block.png)

If we slice the matrix, **we don't have the global sum!**

---

- ###### The Solution: Online Softmax

We can update the Softmax result **incrementally**.
We track local statistics: **Max ($m$)** and **Sum ($l$)**.

![](assets/online-softmax.png)

**Key**: When a new max value is found, shrink the old sum to keep the math correct.

---

The softmax of vector $x\in R^B$ is computed as:

- $m(x) = \underset{i}{\max}\ x_i$
- $f(x) = [e^{x_1 - m(x)}\ \ldots\ e^{x_B - m(x)}]$
- $softmax(x) = \frac{f(x)}{l(x)}$

---

For vectors **$x^{(1)},x^{(2)}\in \mathbf{R}^B$**, we can decompose the softmax of the concatenated **$x = [x^{(1)}, x^{(2)}]\in \mathbf{R}^{2B}$** as:

- $m(x) = \max(m(x^{(1)}), m(x^{(2)}))$
- $f(x) = [e^{m(x^{(1)}) - m(x)}f(x^(1))\ \ e^{m(x^{(2)}) - m(x)}f(x^{(2)})]$
- $l(x) = e^{m(x^{(1)}) - m(x)}l(x^{(1)}) + e^{m(x^{(2)} - m(x))l(x^{(2)})}$

- $P = Softmax(x) = \frac{f(x)}{l(x)}$

---

- ###### Computing the Partial Output

Once we have the Attention Probabilities ($P_{ij}$) for the current block, we multiply them by Value ($V_j$).

$$O_{ij} = P_{ij} \times V_j$$

But we cannot just "Add" this to the previous result, because the **Softmax Denominator** changes!

---

- ###### Rescaling and Merging

To merge the **Old Output** with the **New Partial Output**, we must **Rescale** the old data using the updated Softmax statistics.

![](assets/output.png)

---

**The Update Formula:**

$$
O_{new} = \frac{1}{\ell_{new}} \left( \underbrace{\ell_{old} e^{m_{old} - m_{new}} O_{old}}_{\text{Rescaled Old Output}} + \underbrace{e^{\tilde{m_{ij}} - m_{new}} \tilde{P_{ij}} V}_{\text{New Block Output}} \right)
$$

- **$\tilde{m_{ij}}, \tilde{P_{ij}}$**: $rowmax(S_{ij})$ and $e^{S_{ij} - \tilde{m_{ij}}}$.

> **Meaning**: "Shrink" the old result based on how much the Max value increased, then add the new weighted contribution.

---

```python
# Load Q, K, V from HBM. Split into blocks.
for j in range(Tc):  # Outer Loop: Load K, V blocks (Fits in SRAM)
    K_j, V_j = load_from_HBM(j) 
    for i in range(Tr):  # Inner Loop: Load Q blocks (Fits in SRAM)
        Q_i, O_i, l_i, m_i = load_from_HBM(i)
        # 1. Compute Attention Score
        S_ij = Q_i @ K_j.T
        # 2. Compute Local Softmax Stats
        m_tilde = rowmax(S_ij)
        P_tilde = exp(S_ij - m_tilde)
        l_tilde = rowsum(P_tilde)
        # 3. Update Global Stats (Online Softmax)
        m_new = max(m_i, m_tilde)
        l_new = exp(m_i - m_new) * l_i + exp(m_tilde - m_new) * l_tilde
        # 4. Rescale and Update Output (The Formula)
        O_i = (1 / l_new) * ( 
            l_i * exp(m_i - m_new) * O_i + 
            exp(m_tilde - m_new) * (P_tilde @ V_j) 
        )
        # 5. Write back to HBM
        write_to_HBM(O_i, l_new, m_new)
```
