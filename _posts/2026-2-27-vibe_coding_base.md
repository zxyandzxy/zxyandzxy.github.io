---
layout: post
title: "Vibe Coding 基础"
date: 2026-02-27
tags: [Vibe Coding]
comments: true
author: zxy
---

> 阅读[🌟 AI 编程零基础入门教程 Vibe Coding - 鱼皮的 AI 知识库（免费） - 鱼皮AI导航](https://ai.codefather.cn/library/2010994846520700929)的学习笔记

## 一、Vibe Coding

Vibe Coding（氛围编程）：以自然语言prompt驱动LLM，由AI直接生成并迭代代码的**意图驱动型开发模式**

在这种开发模式下，代码开发由 人 +  AI 共同完成，一共三步：

- 人表达意图：以自然语言讲述需求
- AI实现代码：LLM实现具体代码的编写
- 人+AI协同优化：人测试代码，检查效果，提出改进意见，迭代优化

在这种模式下，人类定位像是有技术基础的产品经理（懂代码，但核心工作是清晰表述需求与迭代改进代码效果），LLM则是程序员（根据需求完成具体的代码编写）

在Vibe Coding的编程模式下，人类的重点在于：有创意的idea（创造力），推导出具体**清晰的需求**以自然语言进行表达（表达能力），最后**检查测试**代码**迭代**优化（产品思维）。所以人类的核心能力在于：能不能总结出有价值的产品功能 + 能不能清晰准确的以自然语言表达出需求

在Vibe Coding的编程模式下，LLM的重点在于：理解人类的prompt，以整体思维构建代码框架，书写准确无误的代码。即Vibe Coding模式下，LLM的编程能力决定了最终产品的质量与开发体验

## 二、Vibe Coding的问题

由于Vibe Coding模式所产出的代码本质上是由LLM输出的，所以LLM的编程能力局限导致了Vibe Coding的上限。就经验来看，有如下问题：

- 界面同质化：AI生成的网站页面都偏向淡紫色或蓝紫渐变，这是由于LLM在预训练时吃这类网站的训练数据比较多。
  - 解决方案是：在prompt中清晰说明自己想要的颜色、风格或直接给出参考案例
- 代码不可控：AI不会对代码的可用性，安全性负责。在大型项目中，LLM由于无法完整理解整个项目的原始构建逻辑，容易出现仅理解部分代码之后直接生成新代码/修改原始代码的行为。这就很容易出现bug，不舍得代码时bug就会一直留存。
  - 尽量让AI生成简单、清晰的代码
  - 每次生成代码之后都要测试
  - 遇到bug就及时回滚，不要抬高沉默成本
  - 打铁还需自身硬，自己懂更多的代码，用AI编程的效率才会更高
- 技术退化的风险：长期只会用AI会导致自己的编程能力下降
  - 不要完全依赖AI编程，自己 + AI 一起编程，AI是自己的编程助手
  - 尝试理解AI生成的代码，看懂之后再使用

## 三、AI编程工具

### 零代码平台

在浏览器中打开即用，不需要懂任何代码，如百度秒哒（[秒哒-无代码应用搭建平台，一句话做应用](https://www.miaoda.cn/)），Lovable（[Lovable - Build Apps & Websites with AI, Fast | No Code App Builder](https://lovable.dev/)）

- 优势：上手快，所见即所得，自动部署
- 局限：功能相对简单，不适合复杂项目

### AI编程IDE

需要下载安装软件，类似于传统的编程IDE + AI编程助手，如Cursor（[Cursor: The best way to code with AI](https://cursor.com/cn)）， Trae([TRAE - The Real AI Engineer | TRAE - The Real AI Engineer](https://www.trae.cn/))，Google Antigravity（[Google Antigravity](https://antigravity.google/)）

- 优势：功能强大，灵活度高，适合大型复杂项目（适合程序员）
- 局限：有学习成本

### 命令行工具

在终端通过命令行直接和LLM对话，如Claude Code（[Claude Code by Anthropic | AI Coding Agent, Terminal, IDE](https://claude.com/product/claude-code)）、Gemini CLI（[Build, debug & deploy with AI | Gemini CLI](https://geminicli.com/)）

- 优势：效率极高、自动化程度强、成本可控
- 局限：需要一定技术基础

## 四、AI模型

按照来源和定位分为三大阵营：

- 国际顶尖LLM：Claude、ChatGPT、Gemini
- 国产优秀LLM：智谱GLM、通义千问、Kimi
- 开源：Llama、Qwen（需要自己服务器部署）

### Claude：编程届最好

Claude 是Anthropic公司推出的Al 模型系列，一直被公认为编程能力最强的Al 模型（在SWE-Bench上得分一直领先，编程领域的SOTA模型）。

Claude4系列主要有两个版本线：Opus是顶配版本，编程能力最强，但速度相对较慢，价格也更高；Sonnet是平衡版本，在性能和速度之间取得了很好的平衡，性价比最高。

Claude Opus 4.6的能力升级：

- 100万token上下文窗口：可以一次性处理超大规模的代码库，不用担心聊着聊着就失忆
- 128K输出 token：一次能生成更长的代码和文档
- 自适应思考：AI会自动判断问题需不需要深度思考，简单问题秒回，复杂问题慢慢想，省时省钱
- 上下文压缩：长时间运行的任务不会因为撞到上下文上限而中断，AI会自动压缩和总结之前的对话

所以Claude适合需要高质量代码的开发者、做复杂项目。其代码的质量非常高。通过购买Cursor Pro，就可以获取。

> Anthropic官方提供的Claude使用技巧与代码示例合集：[anthropics/claude-cookbooks: A collection of notebooks/recipes showcasing some fun and effective ways of using Claude.](https://github.com/anthropics/claude-cookbooks)

### xxxxxxxxxx 我想学习【技术/概念】。请：1）用简单的语言解释它是什么2）给我一个实际的使用例子3）告诉我什么时候应该用它markdown

GPT-5 系列（2025年）：包括通用版本的GPT-5、推理能力更强的GPT-5Pro，以及专门针对逻辑、数学和编程优化的o3版本。

GPT-5.3-Codex（2026年）：专门针对编程场景做了大幅优化，虽然硬编程能力不如Claude，但是**生成代码的速度**比Claude快不少，特别适合需要快速迭代的场景。其次是**知识更新及时**，对最新技术和框架的了解更快。而且**生态更好**，插件和工具支持更丰富，**中文理解和生成能力也更强**。GPT-5.3-Codex还**特别擅长前端开发**，能一次性生成完整度很高的游戏和应用。

### Gemini3.0：超长上下文

Gemini 3.0系列：顶配的Gemini 3 Pro各方面能力都很全面，轻量的Gemini3Flash速度极快，价格也便宜

Gemini3 Pro支持1M Token（约100万字）的输入上下文，也就是一口气记忆大型项目的所有代码 / 超长的对话历史，这样对于大型项目的设计与迭代都很友好，bug出现的可能性低很多

在UI构建方面，Gemini 3 Pro的表现也特别出色。根据实测，它在前端UI设计、3D模型构建等方面的能力很强，在某些场景下甚至超过了 Claude 和 GPT-5。

### 国产模型

- DeepSeek-V3是开源模型，完全免费使用，编程能力在国产模型中数一数二，**API价格极低**，特别适合需要大量调用的场景。
- 阿里通义千问Qwen，在LiveCodeBench测评中的表现甚至超过了GPT-5，**中文理解能力极强**，用中文提需求特别准。
- 智谱GLM-5是清华团队出品的最新模型，2026年2月发布，**全球开源模型综合排名第一**。GLM-5在Coding 和 Agent 能力方面表现非常突出，支持 200K Token 的长上下文，具备强大的工具调用和长程任务规划能力。体感已经接近ClaudeOpus 级别，但作为开源模型，成本要低得多。

### 模型选择

- 做前端／UI项目：Gemini 3 Pro在前端UI设计方面表现特别出色、3D模型构建能力也很强。
- 做全栈项目：优先选择编程能力强的Claude Sonnet，能力全面，前后端都能应对。配合Cursor 使用，开发体验很好。如果需要快速生成完整项目，智谱GLM-5的速度和效果也不错。
- 处理大型代码库：Gemini 3 Pro（1M Token）和Claude Opus 4.6（1M Token）的超长上下文能力最合适，可以一次性分析整个项目。智谱GLM-5支持200KToken，也能处理包含完整前端和后端的中大型项目代码。
- 快速迭代开发：GPT-5的响应速度最快，特别适合需要快速验证想法的场景。智谱GLM在生成速度上也有优势
- 大量测试和调用：DeepSeek完全免费，而且DeepSeek和通义千问的APl 价格极低，适合需要大量调用的场景，测试时可以放心使用。

## 五、AI agent平台

AI 应用开发平台（Dify、Coze、百炼）：可视化配置AI 应用，适合做智能客服、知识库问答等AI应用。通过拖拽的方式配置工作流，设置提示词和知识库，不需要写代码。

AI智能体平台（Flowith、Manus）：Al自主规划和执行复杂任务，可以持续运行几个小时甚至几天。人类只需要描述目标，AI会自己规划步骤、调用工具、执行任务，直到完成为止。相当于，AI智能体平台让AI当项目经理，人类只告诉它目标，它会自己规划和执行。

### 常见的AI agent平台

- Flowith（https://flowith.io/）：无限的步骤、上下文、工具调用，直到完成工具才停止。使用范围可以是：生成网站、制作PPT、编写博客等等。优点在于：无限的执行能力（会自助进行规划、自我根据实际情况调整计划、修正问题，直到最终实现），并行执行能力（可以同时调用多个LLM或工具，支持直接云端部署）。缺点是：执行效率低，Token消耗不太可控，定制能力一般（没办法人为控制中间执行细节）
- Manus（[manus.im](https://www.manus.im/)）：通用AI agent，采用多模型协同机制，具备强大的工具调用能力，能在多个领域自动生成和执行任务。自主规划能力很强，能够独立思考和规划，确保任务的执行
- OpenManus：Manus的开源版本

## 六、AI 编程 IDE

目前来说，感觉最好用的AI IDE还是Cursor

Cursor被称为"Al 时代的VS Code"。因为它基于VS Code 改造，保留了VS Code的所有优点，同时加入了强大的AI功能

- 多种 Al 功能：Tab 代码补全、Agent 智能体、Chat 对话、内联编辑等
- 多模型支持：可以使用Claude、GPT、Gemini、Grok等多个模型，还有Auto自动选择功能
- 上下文感知：Al能理解整个项目的代码，支持最高1MToken的上下文（Max模式）
- 生态成熟：基于VSCode开发，支持所有VSCode 插件

### AI IDE 的使用技巧

1. 善用上下文：代码编辑器最强大的地方在于它能理解整个项目的上下文。要充分利用这一点：

- 在项目根目录创建README.md，描述项目的整体架构
- 使用 .cursorrules（或者其他编辑器支持的规则文件），告诉AI你的编码规范
- 在对话时，引用相关的文件（@文件名）

2. 分步骤实现：不要一次性提出太复杂的需求，而是给出明确的步骤（人想 + AI 优化版本 + 人反思迭代）

3. 熟悉快捷键
   1. Ctrl +L ：切换侧边栏（除非已绑定到某个模式）
   2. Ctrl + I：切换侧边栏（除非已绑定到某个模式）
   3. Ctrl+K：打开行内编辑，可以在当前位置插入AI 生成的代码
   4. Tab：接受补全建议
   5. Ctrl + Shift + L：将选中内容添加到聊天
   6. Alt+⬆️/⬇️：移动当前行
   7. Ctrl+／：注释/取消注释
   8. Ctrl + Shift +F：全局搜索

4. 代码审查：人为审查一下AI生成的代码是否存在问题 / 让另一个LLM来审查当前的代码

5. 善用Git保存开发版本，便于遇到bug回退（勤快提交，一功能/ 一bug就commit一次）

## 七、AI IDE 插件

Cursor是一个独立的编辑器，作为一个完整的软件，但是IDE 插件则是安装在现有IDE上的扩展。

- Cline：最强大的开源AI 插件，支持VS Code和JetBrains全家桶，完全开源免费，支持 Claude、GPT、Gemini、DeepSeek等各种大模型，还可以部署MCP 服务扩展功能。不仅能对话生成代码，还能自主执行命令、修改多个文件、使用浏览器，总之功能非常全面。**（但是国内用户不友好）**
- Claude Code 官方扩展：Claude Code 是 Anthropic 推出的 Al 编程助手，支持VS Code
- GitHub Copilot：目前是最成熟的 Al 编程助手，支持VS Code、JetBrains 全系列，主要是用于代码补全（质量很高），学生免费

### 其他IDE 插件

- GitLens：更直观地查看Git代码的修改历史，把鼠标放到任意代码行上就能看到这行代码的作者、提交时间等信息
- Office Viewer：能在编辑器里直接预览和编辑各种文档，包括Markdown、Excel、Word、PDF等，不用来回切换窗口。
- ESLint是代码质量检查工具，Prettier是代码格式化工具。这两个插件能帮你保持代码规范，避免Al生成的代码出现格式问题
- Error Lens能让错误信息直接高亮显示在代码行尾，一眼就能看到哪里有问题。
- Console Ninja能让你在编辑器里直接看到代码的运行结果，不用频繁切换到浏览器控制台。
