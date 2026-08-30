---
title: 大模型 Prompt 工程实战：从 Zero-Shot 到 ReAct Agent
categories:
  - 大模型
tags:
  - 大模型
  - LLM
  - Prompt
  - Agent
  - RAG
description: 不会写 Prompt 就用不好大模型。本文整理 6 大 Prompt 模式 + ReAct Agent 实战代码，附可复用的模板。
abbrlink: 504f
date: 2024-07-05 09:30:00
---

> 同样的模型，给不同 Prompt 效果天差地别。

本文系统梳理工程中常用的 Prompt 范式，并给出可运行示例。

## 一、Zero-Shot：最简单的开始

直接给任务指令：

```text
把下面这句话翻译成英文：
"深度学习正在重塑软件工程。"
```

适合：分类、改写、翻译、抽取等结构化任务。

## 二、Few-Shot：给例子

模型擅长**模仿格式**，给 2~5 个示例效果立竿见影：

```text
判断情感，仅输出 正面/负面。

示例1: "今天天气真好" -> 正面
示例2: "服务态度太差了" -> 负面
示例3: "还行吧"        -> 正面
"这家咖啡馆性价比超高" ->
```

## 三、Chain-of-Thought (CoT)

让模型"一步一步想"：

```text
Q: 苹果每斤5元，橘子每斤3元。小明买了3斤苹果和4斤橘子，给了50元，找回多少？
A: 让我一步步算：
1) 苹果: 3 × 5 = 15 元
2) 橘子: 4 × 3 = 12 元
3) 总花费: 15 + 12 = 27 元
4) 找回: 50 - 27 = 23 元
答案是 23 元。
```

**关键技巧**：在 prompt 末尾加 `Let's think step by step.` 即可显著提升推理能力。

## 四、Self-Consistency

同一个问题采样 $N$ 次答案，**多数投票**选最优。代价是 N 倍推理开销，但任务越难收益越高。

## 五、ReAct：让模型调用工具

ReAct = **Reasoning + Acting**。模型在思考与工具调用之间交替：

```python
from openai import OpenAI
import json

client = OpenAI()

tools = [
    {
        "type": "function",
        "function": {
            "name": "search",
            "description": "在知识库中检索",
            "parameters": {
                "type": "object",
                "properties": {"query": {"type": "string"}},
                "required": ["query"],
            },
        },
    }
]

def search(query: str) -> str:
    # 真实场景接向量库
    kb = {
        "Transformer": "2017年由 Google 提出的注意力架构...",
        "LoRA": "低秩适配微调方法...",
    }
    return kb.get(query, "未找到相关信息")

def agent(user_query: str, max_iters: int = 5):
    messages = [{"role": "user", "content": user_query}]
    for _ in range(max_iters):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=tools,
        )
        msg = resp.choices[0].message
        messages.append(msg)
        if not msg.tool_calls:
            return msg.content
        for call in msg.tool_calls:
            args = json.loads(call.function.arguments)
            result = search(**args) if call.function.name == "search" else ""
            messages.append({
                "role": "tool",
                "tool_call_id": call.id,
                "content": result,
            })
    return "达到最大迭代次数"

print(agent("请帮我查一下 LoRA 是什么？"))
```

## 六、Structured Output：让输出可解析

```python
from pydantic import BaseModel
from openai import OpenAI

class Sentiment(BaseModel):
    label: str   # positive / negative
    score: float
    keywords: list[str]

client = OpenAI()
resp = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "提取情感与关键词"},
        {"role": "user", "content": "这家店的服务太棒了，强烈推荐！"},
    ],
    response_format=Sentiment,
)
print(resp.choices[0].message.parsed)
# Sentiment(label='positive', score=0.95, keywords=['服务', '棒', '推荐'])
```

## 七、RAG：解决幻觉

RAG (Retrieval-Augmented Generation) 流程：

```
用户问题 → Embedding → 向量库 Top-K → 拼接 Prompt → LLM → 答案
```

关键点：
- **Chunk 切分**：太小召回差，太大噪音多。常用 256~512 token + 10~20% overlap。
- **混合检索**：BM25 + 向量召回，再用 Reranker 精排。
- **引用溯源**：让模型在回答时标 `[1][2]`，可大幅降低幻觉。

## 八、调试技巧清单

1. **先 Zero-Shot 跑通** → 再 Few-Shot → 最后 CoT
2. **看 logprobs**，排查模型"犹豫"的位置
3. **设温度**：分类用 0，创作 0.7~1.0
4. **拆 prompt**：复杂任务拆成多轮对话
5. **用版本管理 prompt**：diff + ab test

## 九、参考资料

- OpenAI Cookbook: [Prompt engineering](https://cookbook.openai.com/examples/techniques_to_improve_reliability)
- Anthropic: [Prompt engineering overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- 论文：*Chain-of-Thought Prompting* (Wei et al., 2022)
- 论文：*ReAct* (Yao et al., 2022)

---

下一篇会基于本文的 ReAct 模板，做一个能查 PDF 知识库的 **RAG Agent**，敬请期待。
