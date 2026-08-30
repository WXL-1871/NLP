---
title: Transformer 架构详解：从 Self-Attention 到多头机制
categories:
  - 深度学习
tags:
  - 深度学习
  - Transformer
  - 注意力机制
  - NLP
mathjax: true
description: Transformer 是当代大模型的基石，本文带你拆解 Self-Attention、多头机制、位置编码与 Layer Norm 的工程细节。
abbrlink: '7769'
date: 2024-06-10 14:00:00
---

> "Attention is All you Need." —— Vaswani et al., 2017

Transformer 彻底改变了 NLP 与 CV。本文不讲八卦，只讲**公式 + 代码**。

## 一、整体架构

```
        ┌────────────────────────┐
        │   Encoder (N=6)        │   ┌──────────────────────┐
Input ─►│ Self-Attn → FFN        ├──►│   Decoder (N=6)      ├──► Output
Embedding│ + Add & Norm × 2      │   │ Masked Self-Attn     │  Embedding
        └────────────────────────┘   │ Cross-Attn → FFN     │
                                     │ + Add & Norm × 3     │
                                     └──────────────────────┘
```

## 二、Self-Attention 公式

给定输入序列 $X \in \mathbb{R}^{n \times d}$，先通过三个可学习矩阵 $W^Q, W^K, W^V$ 投影到三个空间：

$$
Q = X W^Q, \quad K = X W^K, \quad V = X W^V
$$

注意力输出：

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{Q K^\top}{\sqrt{d_k}}\right) V
$$

其中 $\sqrt{d_k}$ 用于防止点积过大导致 softmax 饱和。

### 直观理解

- **Query**："我在找什么"
- **Key**：每个 token 的"标签"
- **Value**：每个 token 实际携带的信息

相似度 $Q K^\top$ 决定每个位置从其他位置"读"多少信息。

## 三、多头注意力

把 $Q, K, V$ 切成 $h$ 份，每份独立做 attention，最后 concat：

$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h) W^O
$$

$$
\text{head}_i = \text{Attention}(Q W_i^Q, K W_i^K, V W_i^V)
$$

多头让不同 head 学习不同子空间的关系（语法、语义、长距离依赖等）。

## 四、位置编码

Self-Attention 是**置换不变**的，必须显式注入位置信息。原始论文用**正弦位置编码**：

$$
PE_{(pos, 2i)}   = \sin\!\left(\frac{pos}{10000^{2i/d}}\right)
$$
$$
PE_{(pos, 2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d}}\right)
$$

现代实现也常用**可学习**的位置编码（BERT）或**旋转位置编码 RoPE**（LLaMA、ChatGLM）。

## 五、Add & Norm

每个子层都做残差 + LayerNorm：

$$
\text{output} = \text{LayerNorm}(x + \text{Sublayer}(x))
$$

工程上常用 **Pre-LN** 变体（GPT / LLaMA 风格）训练更稳定：

$$
x \leftarrow x + \text{Sublayer}(\text{LayerNorm}(x))
$$

## 六、PyTorch 极简实现

```python
import torch
import torch.nn.functional as F
from torch import nn

class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        assert d_model % n_heads == 0
        self.d_k = d_model // n_heads
        self.n_heads = n_heads
        self.W_qkv = nn.Linear(d_model, 3 * d_model)
        self.W_o   = nn.Linear(d_model, d_model)

    def forward(self, x, mask=None):
        B, L, D = x.shape
        qkv = self.W_qkv(x).reshape(B, L, 3, self.n_heads, self.d_k)
        q, k, v = qkv.permute(2, 0, 3, 1, 4)  # (3, B, H, L, d_k)
        scores = (q @ k.transpose(-2, -1)) / (self.d_k ** 0.5)  # (B, H, L, L)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float("-inf"))
        attn = F.softmax(scores, dim=-1)
        out = (attn @ v).transpose(1, 2).reshape(B, L, D)
        return self.W_o(out)
```

## 七、为什么 Transformer 这么强

1. **并行**：相比 RNN 没有时序依赖，GPU 友好。
2. **长距离依赖**：self-attention 直接连接任意两个位置，路径长度 $O(1)$。
3. **可扩展**：堆叠 N 层就能表达更复杂关系；规模上去后出现**涌现**。

## 八、练手项目

- [nanoGPT](https://github.com/karpathy/nanoGPT)：100 行复现 GPT
- [minimind](https://github.com/jingyaogong/minimind)：从零预训练一个 26M 小模型
- [annotated-transformer](http://nlp.seas.harvard.edu/annotated-transformer/)：哈佛 NLP 注释版

---

下一篇文章会用上面的知识，从零训练一个**微型 GPT**，敬请期待。
