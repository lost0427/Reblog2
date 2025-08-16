
---
title: 'MCP 所需的大语言模型实测对比'
published: 2025-04-15
draft: false
description: '首次尝试MCP'
tags: ['ai']
---

# MCP 所需的大语言模型

在 **MCP（Model Context Protocol）** 的使用中，并非所有大语言模型都能很好地胜任。目前可免费使用的大语言模型可以通过 [OpenRouter](https://openrouter.ai/) 获取。

从实践来看，只有较高级的模型才能更好地配合 MCP。下面这张来自 Cursor 的模型支持图可以作为参考：

![请检测网络连接](https://cdn.jsdelivr.net/gh/lost0427/Reblog@main/public/Tech/ai-mcp-0/cursor-0.webp "来自 Cursor 的模型支持图")

<small>有趣的是，在这张图中最便宜的竟然是 OpenAI 的 o3-mini 😅</small>

---

## 模型实际使用体验

* **DeepSeek R1 满血版（671B）**
  实测完全可以调用 MCP，表现稳定。

* **阿里云 Llama-R1-Distill-70B**
  平时对话体验还行，但在 MCP 场景下能力不足。

* **Gemini-2.5-Pro-Exp-03-25**
  可在 Google AI 官网免费使用，每天提供 1500 次调用额度。
  同时，OpenRouter 上也能找到 Gemini 2.5 Pro 版本。

* **OpenRouter 免费模型**
  包括 `nvidia/llama-3.1-nemotron-ultra-253b-v1:free` 以及 `deepseek/deepseek-r1:free`，均可用于 MCP。

需要注意的是，**模型调用 MCP 的能力并不是简单的 0/1 开关**。有些模型更“愿意”调用工具，或者说更“清楚”在什么场景下需要调用工具。

---

## 自建接口的尝试

为了降低成本并提高响应率，我尝试开发了一个 **OpenAI 兼容接口**，可调用 [Duck.ai](https://duck.ai) 上的所有模型。

* DuckAI 的 **o3-mini** 上下文长度较短，日常使用没问题，但在通过 `cline` 调用时会报错。
* 该接口未做鉴权，仅供学习和研究使用，有需要的朋友可以私信获取。

---

## 游戏场景下的应用

为了让大语言模型在游戏设定中更自然，可以使用 **Silly Tavern** 的角色卡和世界书功能，帮助模型补充：

* 角色设定
* 世界观与专有名词解释

相关资源：

* [Silly Tavern GitHub](https://github.com/SillyTavern/SillyTavern)
* 中文社区（类脑社区）：[Discord 邀请链接](https://discord.com/app/invite-with-guild-onboarding/dNfheFUe9M)

（注：服务并非常开，相关账号与链接可在群内获取）

---

## 技术资料整理

以下是一些与 MCP 和大语言模型相关的技术链接：

* **MCP 技术文档**

  * [MCP 简介](https://mcp-docs.cn/introduction)
  * [SSE 模式说明](https://mcp-docs.cn/docs/concepts/transports#sse)
  * [官方规范文档](https://model-context-protocol.github.io/specification/basic/transports/)
  * [OpenAI MCP SSE 示例](https://github.com/openai/openai-agents-python/tree/main/examples/mcp/sse_example)

* **超长上下文模型**

  * [Qwen Agent](https://github.com/QwenLM/Qwen-Agent/blob/main/README_CN.md)

* **语音识别**

  * [Whisper X](https://github.com/m-bain/whisperX)
  * [Faster-Whisper](https://github.com/SYSTRAN/faster-whisper)

* **低成本多模态模型**

  * [Qwen2.5-Omni-7B (HuggingFace)](https://huggingface.co/Qwen/Qwen2.5-Omni-7B)
  * [Qwen2.5-Omni GitHub](https://github.com/QwenLM/Qwen2.5-Omni/blob/main/README_CN.md)

