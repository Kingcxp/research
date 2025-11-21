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

<!-- _header: NLP (Nature Language Processing) -->

- ###### What does NLP want to solve?

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### What does NLP want to solve?

The application for NLP is **translation**.

In translation, we want to translate a sentence to another sentence.

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### What does NLP want to solve?

The application for NLP is **translation**.

In **translation**, we want to translate a sentence to another sentence.

The sentence and the translation result can be **anything**, for example:

![](assets/translate-example.png)

To put it more vividly, given the sentence as context, we need to predict what's next.

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### How to represent words so that computers can **understand** them?

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### How to represent words so that computers can **understand** them?
  
- **Solution**: Represent words as **vectors** (embeddings)

![](assets/embedding.png)

- Through the help of a pretrained matrix, each token is converted into a vector of size $d_{model}$, which every number in the vector represents a sematic feature of the token.

---

<!-- _header: Before the Transformers -->

- ###### So before the **Transformers**, how did we translate the embeddings?

