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
---

<!-- _class: title-page -->

## Overview of LLM:
## **Reasoning** and **Optimization**

---

<!-- _header: Catalog -->

- #### Transformer
  - What does NLP want to solve
  - Before transformer
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

The most common application of NLP is **translation**.

In translation, we want to translate a sentence to another sentence.

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### What does NLP want to solve?

The most common application of NLP is **translation**.

In translation, we want to translate a sentence to another sentence.

The sentence and the translation result can be **anything**, for example:

[![](https://mermaid.ink/img/pako:eNo9jUtvgzAQhP-KtWeCjI0x8SFVH4ceeqh6bMhhhc1DARs5RmlL-O81kdrVHHZ29O0sUDttQEEzuGvdoQ_k7aOyJM7j8dVdSXCkdu5M0BKcpsE8nMhud7gFj_YyYDA38nR8nwPpoyxpem_S08ZDAq3vNajgZ5PAaPyIm4VlSysInRlNBSquGv25gsqukZnQfjo3_mHezW0HqsHhEt086dj40mPrcfy_emO18c9utgFUKeT9CagFvkBxkWZciKKQWZYzIcsEvkExztNCliLndE9LRvmawM-9laZSMCoZ3ZeUsSzP-PoLhbRYuQ?type=png)](https://mermaid.live/edit#pako:eNo9jUtvgzAQhP-KtWeCjI0x8SFVH4ceeqh6bMhhhc1DARs5RmlL-O81kdrVHHZ29O0sUDttQEEzuGvdoQ_k7aOyJM7j8dVdSXCkdu5M0BKcpsE8nMhud7gFj_YyYDA38nR8nwPpoyxpem_S08ZDAq3vNajgZ5PAaPyIm4VlSysInRlNBSquGv25gsqukZnQfjo3_mHezW0HqsHhEt086dj40mPrcfy_emO18c9utgFUKeT9CagFvkBxkWZciKKQWZYzIcsEvkExztNCliLndE9LRvmawM-9laZSMCoZ3ZeUsSzP-PoLhbRYuQ)

To put it more vividly, given the sentence as context, we need to predict what's next.

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### How to represent words so that computers can **understand** them?

---

<!-- _header: NLP (Nature Language Processing) -->

- ###### How to represent words so that computers can **understand** them?
  
- **Solution**: Represent words as **vectors** (embeddings)

[![](https://mermaid.ink/img/pako:eNpVkLtugzAUhl_FOguLY_mCA2WolIQFqVPHAoMbHIIS7MiFXhLy7rVDm6pnsc_v7_PtAlvbaMhgd7Qf271yA3p6rgzytSoLdLTvGq2KGi0Wj9NgD9p0Zz2hdVlBGRURRlEgwrgqorqCelbXgUezgAaLinxCm-AwihGnEiNB_-jNTBd5QLV51U3TmXZCeTBKSjjDaEFJ7DVCCEaUCFHj2Q1V-sVlEnLG7wiV_xBKUq_7PI5_EC8lrP69BWBoXddANrhRY-i161Vo4RJWKxj2utcVZH7aKHeooDJX75yUebG2_9WcHds9ZDt1fPPdeGrUoPNOtU7199Rp02i3saMZIJOM3TaB7AKfkHFKpJBpwiSViUiE_yn48rFIyZImXMaUp4JTccVwvh3rn8UYZ4w-8JhR5q3rN6GNfc8?type=png)](https://mermaid.live/edit#pako:eNpVkLtugzAUhl_FOguLY_mCA2WolIQFqVPHAoMbHIIS7MiFXhLy7rVDm6pnsc_v7_PtAlvbaMhgd7Qf271yA3p6rgzytSoLdLTvGq2KGi0Wj9NgD9p0Zz2hdVlBGRURRlEgwrgqorqCelbXgUezgAaLinxCm-AwihGnEiNB_-jNTBd5QLV51U3TmXZCeTBKSjjDaEFJ7DVCCEaUCFHj2Q1V-sVlEnLG7wiV_xBKUq_7PI5_EC8lrP69BWBoXddANrhRY-i161Vo4RJWKxj2utcVZH7aKHeooDJX75yUebG2_9WcHds9ZDt1fPPdeGrUoPNOtU7199Rp02i3saMZIJOM3TaB7AKfkHFKpJBpwiSViUiE_yn48rFIyZImXMaUp4JTccVwvh3rn8UYZ4w-8JhR5q3rN6GNfc8)

- Through the help of a pretrained matrix, each token is converted into a vector of size $d_{model}$, which every number in the vector represents a sematic feature of the token.
