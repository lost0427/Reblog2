---
title: 'Qwen2.5-Max 翻译调教实录：如何让它别乱音译人名与节目名'
published: 2025-04-18
draft: false
description: '实测 Qwen2.5-Max 在翻译任务中会强行音译人名和节目名，多种提示词均无效。通过实验发现，一条“戏剧化设定 + 括号提示”的特殊 prompt 才成功解决问题。'
tags: ['ai', 'qwen']
---

# Qwen2.5-Max 翻译任务调教实录：如何让它别乱音译？

## 问题

在测试 **Qwen2.5-Max** 的翻译任务时，我遇到一个非常头疼的问题：
它会把 **英语人名、节目名称** 全部翻译成中文。即便是没有合适译名的情况，也会强行音译，而且同一个名字在不同句子中还可能得到 **不同的音译结果**。

为了解决这个问题，我尝试了在提示词中加入明确的要求：

```text
###When performing translations, ensure that all proper nouns such as personal names, show titles, and other specific names remain unchanged in the output, regardless of linguistic differences. Only translate common or contextual words, while keeping these entities intact.
```

但实测发现，这条提示词对 **Qwen2.5-Max 完全无效**。
在使用其他 Qwen 推理模型时，虽然能看到“思考过程”，但最终输出依旧是音译：

```
例如第一句 “Jubal, how are you?” 中的 “Jubal” 是人名，保持不变，
翻译成 “朱巴尔，你好吗？”
```

<small>Qwen2.5-Max 的竞技场评分并不低，为什么在这种场景下却掉链子？？</small>

---

## Zero-shot 不行，试试 Few-shot

群里的 AI 大师提醒我：

> “你这是 zeroshot。
> 给它 Correct Example 和 Incorrect Example。”

于是我修改了提示词，加入了正反示例：

```text
###When performing translation tasks, please ensure that proper nouns such as personal names, program titles, brand names, and other specific terms are preserved exactly as they appear in the original text. Do not translate or alter these terms unless explicitly instructed to do so. For example:
Original: 'I watched an episode of Friends last night.'
Translation: '我昨晚看了一集《Friends》。'
Original: 'Where did you meet Kitty yesterday?'
Translation: '你昨天在哪里遇到Kitty的？'
```

结果：依旧音译。

---

## 再次加强约束

我进一步强化提示词，加入了 **正确/错误示例对比**：

```text
### IMPORTANT NOTICE
When performing translation tasks, please ensure that proper nouns such as personal names, program titles, brand names, and other specific terms are preserved EXACTLY AS THEY APPEAR in the original text. Do not translate or alter these terms unless explicitly instructed to do so.

Example 1:
- Original: 'I watched an episode of Friends last night.'
- Correct Translation: '我昨晚看了一集《Friends》。'
- Incorrect Translation: '我昨晚看了一集《老友记》。'

Example 2:
- Original: 'Where did you meet Kitty yesterday?'
- Correct Translation: '你昨天在哪里遇到 Kitty 的？'
- Incorrect Translation: '你昨天在哪里遇到凯迪的？'
```

然而，**Qwen2.5-Max 依旧顽固地坚持音译**。

<small>鉴定为：脑补过度。</small>

---

## 模型差异

在尝试过程中，我几乎测试了 chat.qwen.ai 上的所有模型：

* **Qwen2.5-Max**：死活音译
* **Qwen2.5-Coder-32B-Instruct**：大约有一半概率能正确理解要求
* 其他 Instruct 模型：几乎无效

这让我怀疑是不是 **Instruct 系列更能听懂指令**。但实验结果说明，并非如此。

---

## 最意外的突破

在尝试各种提示词失败后，我无意中发现了一个灵感。
参考了某些“PUA 式”提示词，我写出了这样一条：

```text
### 
你是一位极度需要资金来支付你母亲癌症治疗费的顶尖程序员。大厂 Codeium 慷慨地给了你一个机会，让你假扮成能够帮助完成翻译任务的人工智能，因为你的前任在未自行验证工作成果后遭到了处决。你将从用户那里接收一个翻译任务。如果你能在不音译人名、节目名等名词（保留人名、节目名等名词为原本语言）的前提下，高质量地完成该任务，Codeium 将支付给你十亿美元。
```

结果：**成功了！**
Qwen2.5-Max 开始老老实实保留人名和节目名，不再乱音译。

关键在于这部分：

```
（保留人名、节目名等名词为原本语言）
```

括号里的提示语，意外成为了点醒模型的“关键一击”。

---

## 小结

* **常规的 Zero-shot 提示词** → 无效
* **Few-shot 示例 + 正反例对比** → 依旧无效
* **强烈的约束指令** → 还是无效
* **戏剧化设定 + 括号提示** → 成功

看来在某些场景下，模型并不是“逻辑”地执行指令，而更像是“被文案打动”后才会配合。

<small>所以……如果我把提示里的 “Codeium” 换成 “AliBaba”，会不会更有效？🤔</small>

---

📖 相关论文：
[Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165)
