---
layout: post
title: "LLM整体学习路径"
date: 2025-06-14
tags: [LLM]
comments: true
author: zxy
---

## 课程体系

1. LLM基础：包括数学基础、python编程基础、神经网络基础
2. LLM底层机理：包括构建最优LLM的各类前沿技术
3. LLM工程实践：包括如何创建基于LLM的应用程序并部署LLM

> 学习参考资料：[mlabonne/llm-course: 课程，帮助你入门大型语言模型（LLMs），包含路线图和 Colab 笔记本。 --- mlabonne/llm-course: Course to get into Large Language Models (LLMs) with roadmaps and Colab notebooks.](https://github.com/mlabonne/llm-course)

## LLM基础

![img](https://github.com/mlabonne/llm-course/raw/main/img/roadmap_fundamentals.png)

### Mathematics for Machine Learning

LLM本质是一个神经网络搭建的模型，神经网络又是机器学习的一种特殊实现。机器学习底层算法需要依靠各种数学概念进行支撑。具体来说，这些数学知识有：

- 线性代数：这对于理解许多算法至关重要，尤其是在深度学习中使用的算法。关键概念包括向量、矩阵、行列式、特征值和特征向量、向量空间和线性变换。
- 微积分：许多机器学习算法涉及连续函数的优化，这需要理解导数、积分、极限和级数。多元微积分和梯度概念也很重要。
- 概率论与数理统计：这对于理解模型如何从数据中学习并做出预测至关重要。关键概念包括概率论、随机变量、概率分布、期望、方差、协方差、相关系数、假设检验、置信区间、最大似然估计和贝叶斯推理。

> 数学知识的学习资源

- [3Blue1Brown - The Essence of Linear Algebra](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab): 讲解线性代数的本质，通过几何直观理解线性代数中的各种概念
- [StatQuest with Josh Starmer - Statistics Fundamentals](https://www.youtube.com/watch?v=qBigTkBLU6g&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9): 统计学基础：为许多统计学概念提供简单清晰的解释。
- [AP Statistics Intuition by Ms Aerin](https://automata88.medium.com/list/cacc224d5e7d):  Aerin 老师提供的文章列表，这些文章为每种概率分布提供了直觉理解。
- [Immersive Linear Algebra](https://immersivemath.com/ila/learnmore.html): 线性代数的另一种视觉解释。
- [Khan Academy - Linear Algebra](https://www.khanacademy.org/math/linear-algebra): 非常适合初学者，因为它以非常直观的方式解释了线性代数的概念。
- [Khan Academy - Calculus](https://www.khanacademy.org/math/calculus-1): 一个涵盖微积分所有基础的互动课程。
- [Khan Academy - Probability and Statistics](https://www.khanacademy.org/math/statistics-probability): 以易于理解的方式呈现概率统计的知识。

### Python for Machine Learning

python作为一种功能强大且灵活的编程语言，已经成为机器学习领域的流行编程语言，具备一系列支撑各种算法编写的库，其面向机器学习的具体知识有：

- Python 基础：Python 编程需要深入理解基本语法、数据类型、错误处理和面向对象编程。
- 数据科学库：包括熟悉 NumPy 进行数值运算，Pandas 进行数据操作和分析，Matplotlib 和 Seaborn 进行数据可视化。
- 数据预处理：这涉及特征缩放和归一化，处理缺失数据，异常值检测，分类数据编码，以及将数据分割为训练集、验证集和测试集。
- 机器学习库：熟练使用 Scikit-learn 这一提供丰富监督学习和无监督学习算法的库至关重要。理解如何实现线性回归、逻辑回归、决策树、随机森林、k 近邻（K-NN）和 K 均值聚类等算法很重要。主成分分析（PCA）和 t-SNE 等降维技术也有助于可视化高维数据。

> python基础学习资源

- [Real Python](https://realpython.com/): 一个全面的资源，包含针对初学者和高级 Python 概念的教程和文章。
- [freeCodeCamp - Learn Python](https://www.youtube.com/watch?v=rfscVS0vtbw): 一个长视频，全面介绍了 Python 的所有核心概念。
- [Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/): 一本免费的数字书籍，是学习 pandas、NumPy、Matplotlib 和 Seaborn 的绝佳资源。
- [freeCodeCamp - Machine Learning for Everybody](https://youtu.be/i_LwzRVP7bg): Practical introduction to different machine learning algorithms for beginners.
- [Udacity - Intro to Machine Learning](https://www.udacity.com/course/intro-to-machine-learning--ud120): 机器学习入门：免费课程，涵盖 PCA 和其他机器学习概念。

###  Neural Networks 

神经网络是许多机器学习模型的基本组成部分，特别是在深度学习领域。要有效地使用它们，必须全面了解其设计和机制，具体知识点包括：

- 基础知识：这包括理解神经网络的结构，如层、权重、偏差和激活函数（Sigmoid、Tanh、ReLU 等）。
- 训练与优化：熟悉反向传播和不同类型的损失函数，如均方误差（MSE）和交叉熵。了解各种优化算法，如梯度下降、随机梯度下降、RMSprop 和 Adam。
- 过拟合：理解过拟合的概念（即模型在训练数据上表现良好但在未见数据上表现不佳），并学习各种正则化技术（dropout、L1/L2 正则化、提前停止、数据增强）来防止过拟合。
- 实现多层感知器（MLP）：使用 PyTorch 构建一个多层感知器（MLP），也称为全连接网络。

> 神经网络的学习资源

- [3Blue1Brown - But what is a Neural Network?](https://www.youtube.com/watch?v=aircAruvnKk): 这个视频直观地解释了神经网络及其内部工作原理。
- [freeCodeCamp - Deep Learning Crash Course](https://www.youtube.com/watch?v=VyWAvY2CF9c): 这个视频高效地介绍了深度学习中的所有最重要概念。
- [Fast.ai - Practical Deep Learning](https://course.fast.ai/): 为有编程经验并想学习深度学习的人设计的免费课程。
- [Patrick Loeber - PyTorch Tutorials](https://www.youtube.com/playlist?list=PLqnslRFeH2UrcDBWF5mfPGpqQDSta6VK4): 一系列视频，供完全初学者学习 PyTorch。

### Natural Language Processing (NLP)

自然语言处理 (NLP) 是人工智能的一个引人入胜的分支，它连接了人类语言和机器理解。从简单的文本处理到理解语言细微差别，NLP 在翻译、情感分析、聊天机器人等许多应用中发挥着关键作用。

- 文本预处理：学习各种文本预处理步骤，如分词（将文本分割成单词或句子）、词干提取（将单词还原为其基本形式）、词形还原（与词干提取类似，但考虑上下文）、停用词移除等。
- 特征提取技术：熟悉将文本数据转换为机器学习算法可理解格式的技术。关键方法包括词袋模型（BoW）、词频-逆文档频率（TF-IDF）和 n-gram。
- 词嵌入：词嵌入是一种词表示方法，它允许具有相似含义的词具有相似的表示。主要方法包括 Word2Vec、GloVe 和 FastText。
- 循环神经网络（RNNs）：理解循环神经网络的工作原理，这是一种专为处理序列数据而设计的神经网络。探索 LSTMs 和 GRUs，这两种能够学习长期依赖关系的 RNN 变体。

> NLP学习资源

- [Lena Voita - Word Embeddings](https://lena-voita.github.io/nlp_course/word_embeddings.html): 关于词嵌入相关概念的入门课程。
- [RealPython - NLP with spaCy in Python](https://realpython.com/natural-language-processing-spacy-python/): 关于 Python 中用于 NLP 任务的 spaCy 库的详尽指南。
- [Kaggle - NLP Guide](https://www.kaggle.com/learn-guide/natural-language-processing): 一些用于 Python 中 NLP 实践解释的笔记本和资源。
- [Jay Alammar - The Illustration Word2Vec](https://jalammar.github.io/illustrated-word2vec/):  理解著名 Word2Vec 架构的良好参考。
- [Jake Tae - PyTorch RNN from Scratch](https://jaketae.github.io/study/pytorch-rnn/): PyTorch 中 RNN、LSTM 和 GRU 模型的实用简单实现。
- [colah's blog - Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/): 一篇更理论性的关于 LSTM 网络的文章。

## LLM底层机理

![img](https://github.com/mlabonne/llm-course/raw/main/img/roadmap_scientist.png)

### The LLM architecture 

对 Transformer 架构的深入了解不是必需的，但理解现代 LLMs 的主要步骤很重要：通过 tokenization 将文本转换为数字，通过包含注意力机制的层处理这些标记，最后通过各种采样策略生成新文本。

- 架构概述：了解从编码器-解码器 Transformer 到像 GPT 这样的仅解码器架构的演变，这些架构构成了现代 LLMs 的基础。重点关注这些模型在高级别上如何处理和生成文本。
- Tokenization：学习 Tokenization 的原理——文本如何被转换为 LLMs 可以处理的数值表示。探索不同的分词策略及其对模型性能和输出质量的影响。
- 注意力机制：掌握注意力机制的核心概念，特别是自注意力和其变体。理解这些机制如何使 LLMs 能够处理长距离依赖关系并在整个序列中保持上下文。
- 采样技术：探索各种文本生成方法及其权衡。比较确定性方法（如greedy search和beam search）与概率方法（如temperature sampling和nucleus sampling）。

> LLM架构学习资源

- [Visual intro to Transformers](https://www.youtube.com/watch?v=wjZofJX0v4M)： 3Blue1Brown 制作的 Transformer 视觉入门介绍，适合完全初学者。
- [LLM Visualization](https://bbycroft.net/llm)：rendan Bycroft 制作的交互式 3D LLM 内部结构可视化
- [nanoGPT](https://www.youtube.com/watch?v=kCc8FmEb1nY)：一个 2 小时长的 YouTube 视频，用于从头开始重新实现 GPT（面向程序员）。他还制作了一个关于 [tokenization](https://www.youtube.com/watch?v=zduSFxRajkE) 的视频。
- [Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/)：历史概述介绍注意力机制的需求。
- [Decoding Strategies in LLMs](https://mlabonne.github.io/blog/posts/2023-06-07-Decoding_strategies.html) ：提供代码和视觉介绍，解释不同的解码策略以生成文本。

### Pre-training models

预训练是一个计算密集且成本高昂的过程。虽然这不是本课程的重点，但了解模型如何进行预训练非常重要，特别是在数据和参数方面。

- 数据准备：预训练需要庞大的数据集（例如，Llama 3.1 是在 150 万亿个 token 上训练的），这些数据集需要进行仔细的筛选、清理、去重和 tokenization 。现代预训练流程实现了复杂的过滤机制，以去除低质量或存在问题的内容。
- 分布式训练：结合不同的并行化策略：数据并行（批处理分发）、流水线并行（层分发）和张量并行（操作拆分）。这些策略需要优化 GPU 集群间的网络通信和内存管理。
- 训练优化：使用带 warm-up 的学习率自适应、梯度裁剪和归一化来防止爆炸，采用混合精度训练以提高内存效率，以及使用经过调优超参数的现代优化器（AdamW、Lion）。
- 监控：使用仪表板跟踪关键指标（损失、梯度、GPU 状态），为分布式训练问题实施定向日志记录，并设置性能分析以识别跨设备计算和通信中的瓶颈。

> 预训练LLM的学习资源

- [FineWeb](https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1)：文章介绍如何为 LLM 预训练重建大规模数据集（15T），包括高质量子集 FineWeb-Edu。
- [RedPajama v2](https://www.together.ai/blog/redpajama-data-v2)：一篇关于大规模预训练数据集的文章和论文，包含许多有趣的质量过滤器。
- [nanotron](https://github.com/huggingface/nanotron)：极简的 LLM 训练代码库，用于创建 SmolLM2。
- [Parallel training](https://www.andrew.cmu.edu/course/11-667/lectures/W10L2 Scaling Up Parallel Training.pdf)：优化和并行技术概述。
- [Distributed training](https://arxiv.org/abs/2407.20018)：关于在分布式架构上高效训练 LLM 的研究。
- [OLMo 2](https://allenai.org/olmo)：开源语言模型，包含模型、数据、训练和评估代码。
- [LLM360](https://www.llm360.ai/)：一个开源 LLM 框架，包含训练和数据准备的代码、数据、指标和模型。

### Post-training datasets

后训练 的 数据集具有精确的结构，包含指令和答案（用于监督微调）或指令和选择/拒绝的答案（用于偏好对齐）。对话结构比用于预训练的原始文本要罕见得多，这就是为什么我们经常需要处理种子数据并对其进行细化，以提高样本的准确性、多样性和复杂性。更多信息和示例可以在我的仓库 [💾 LLM Datasets](https://github.com/mlabonne/llm-datasets) 中找到。

- 存储与聊天模板：由于对话结构，后训练数据集以特定格式存储，如 ShareGPT 或 OpenAI/HF。随后，这些格式映射到聊天模板如 ChatML 或 Alpaca，以生成模型训练的最终样本。
- 合成数据生成：基于种子数据，使用前沿模型如 GPT-4o 创建指令-响应对。这种方法允许灵活且可扩展地创建高质量答案的数据集。关键考量包括设计多样化的种子任务和有效的系统提示。
- 数据增强：使用验证输出（通过单元测试或求解器）、拒绝采样生成多个答案、 [Auto-Evol](https://arxiv.org/abs/2406.00770)、思维链、分支-求解-合并、角色扮演等技术来增强现有样本。
- 质量过滤：传统技术包括基于规则的过滤、删除重复或近似重复（使用 MinHash 或嵌入）以及 n-gram 去污染。奖励模型和评判 LLMs 通过细粒度和可定制的质量控制补充这一步骤。

> 后训练数据生成 学习资料

- [Synthetic Data Generator](https://huggingface.co/spaces/argilla/synthetic-data-generator)：在 Hugging Face 空间中，使用自然语言构建数据集的初学者友好方法。
- [LLM Datasets](https://github.com/mlabonne/llm-datasets)：用于后训练处理的精选数据集和工具列表。
- [NeMo-Curator](https://github.com/NVIDIA/NeMo-Curator)：用于预训练和后训练数据 的 数据集准备和整理框架。
- [Distilabel](https://distilabel.argilla.io/dev/sections/pipeline_samples/)：用于生成合成数据的框架。它还包括像 UltraFeedback 这样的论文的有趣复现。
- [Semhash](https://github.com/MinishLab/semhash)：用于近似去重和去污染的极简库，配有精炼的嵌入模型。
- [Chat Template](https://huggingface.co/docs/transformers/main/en/chat_templating)：Hugging Face 关于聊天模板的文档。

### Supervised Fine-Tuning

有监督微调（SFT）将基础模型转化为能够回答问题和遵循指令的有用助手。在这个过程中，它们学习如何组织答案并重新激活在预训练期间学习到的一小部分知识。灌输新知识是可能的，但比较肤浅：它不能用来学习一门全新的语言。始终优先考虑数据质量而非参数优化。

- 训练技术：全量微调会更新所有模型参数，但需要大量的计算资源。参数高效的微调技术如 LoRA 和 QLoRA 通过训练少量适配器参数来减少内存需求，同时保持基础权重冻结。QLoRA 结合了 4 位量化与 LoRA 来减少 VRAM 使用。这些技术都在最流行的微调框架中实现：TRL、Unsloth 和 Axolotl。
- 训练参数：关键参数包括带调度器的学习率、批大小、梯度累积、训练轮数、优化器（如 8 位 AdamW）、正则化的权重衰减以及用于训练稳定的预热步数。LoRA 还增加了三个参数：秩（通常为 16-128）、alpha（秩的 1-2 倍）和目标模块。
- 分布式训练：使用 DeepSpeed 或 FSDP 在多个 GPU 上扩展训练。DeepSpeed 通过状态分区提供三种 ZeRO 优化阶段，具有逐步提高的内存效率。这两种方法都支持梯度检查点以实现内存效率。
- 监控：跟踪训练指标，包括损失曲线、学习率调度和梯度范数。监控常见问题，如损失峰值、梯度爆炸或性能退化。

> 有监督微调的 学习资料

- [Fine-tune Llama 3.1 Ultra-Efficiently with Unsloth](https://huggingface.co/blog/mlabonne/sft-llama3)：关于如何使用 Unsloth 微调 Llama 3.1 模型的实践教程。
- [Axolotl - Documentation](https://axolotl-ai-cloud.github.io/axolotl/)：包含大量与分布式训练和数据集格式相关的有趣信息。
- [Mastering LLMs](https://parlance-labs.com/education/)：关于微调（但也包括 RAG、评估、应用和提示工程）的教育资源集合。
- [LoRA insights](https://lightning.ai/pages/community/lora-insights/)：关于 LoRA 的实用见解以及如何选择最佳参数。

### Preference Alignment

偏好对齐是训练后流程中的第二阶段，专注于将生成的答案与人类偏好对齐。该阶段旨在调整 LLMs 的语气，减少毒性和幻觉。然而，提高其性能和改善实用性也变得越来越重要。与 SFT 不同，偏好对齐算法有很多种。这里我们将重点关注最重要的三种：DPO、GRPO 和 PPO。

- 拒绝采样：对于每个prompt，使用训练好的模型生成多个响应，并对它们进行评分以推断选择/拒绝的答案。这会创建符合策略的数据，其中两个响应都来自正在训练的模型，从而提高对齐稳定性。
- **[Direct Preference Optimization](https://arxiv.org/abs/2305.18290)**：直接优化策略，以最大化被选中响应的似然度相对于被拒绝响应。它不需要奖励建模，这使得它在计算效率上比强化学习技术更高，但在质量上略差。非常适合创建聊天模型。
- **Reward model**：使用人类反馈训练奖励模型以预测人类偏好等指标。它可以利用 [TRL](https://huggingface.co/docs/trl/en/index), [verl](https://github.com/volcengine/verl), and [OpenRLHF](https://github.com/OpenRLHF/OpenRLHF) 等框架进行可扩展的训练。
- 强化学习：强化学习技术如 [GRPO](https://arxiv.org/abs/2402.03300) and [PPO](https://arxiv.org/abs/1707.06347) 通过迭代更新策略以最大化奖励，同时保持与初始行为的接近。它们可以使用奖励模型或奖励函数来评分响应。它们通常计算成本高，需要仔细调整超参数，包括学习率、批处理大小和裁剪范围。非常适合创建推理模型。

> 偏好对齐的 学习资料

- [Illustrating RLHF](https://huggingface.co/blog/rlhf)： 使用奖励模型训练和强化学习的微调 RLHF 介绍。
- [LLM Training: RLHF and Its Alternatives](https://magazine.sebastianraschka.com/p/llm-training-rlhf-and-its-alternatives)：RLHF 过程的概述和替代方法如 RLAIF。
- [Preference Tuning LLMs](https://huggingface.co/blog/pref-tuning)：比较 DPO、IPO 和 KTO 算法以执行偏好对齐。
- [Fine-tune with DPO](https://mlabonne.github.io/blog/posts/Fine_tune_Mistral_7b_with_DPO.html)：使用 DPO 微调 Mistral-7b 模型并复现 [NeuralHermes-2.5](https://huggingface.co/mlabonne/NeuralHermes-2.5-Mistral-7B)
- [Fine-tune with GRPO](https://huggingface.co/learn/llm-course/en/chapter12/5)：一个使用 GRPO 微调小模型的实际练习
- [DPO Wandb logs](https://wandb.ai/alexander-vishnevskiy/dpo/reports/TRL-Original-DPO--Vmlldzo1NjI4MTc4)：它展示了你需要追踪的主要 DPO 指标以及你应该预期的趋势。

### Evaluation

可靠地评估 LLMs 是一项复杂但至关重要的任务，它指导着数据生成和训练。它提供了关于改进领域的宝贵反馈，这些反馈可以被用来修改数据混合、质量和训练参数。然而，始终要记住 Goodhart 定律：“当一个指标成为目标时，它就不再是好指标了。”

- **Automated benchmarks**：使用精选数据集和指标（如 MMLU）在特定任务上评估模型。它适用于具体任务，但在抽象和创造性能力方面表现不佳。此外，它也容易受到数据污染。
- **Human evaluation**：它涉及人类提示模型并对响应进行评分。方法范围从简单的情绪检查到遵循特定指南的系统化标注以及大规模社区投票（竞技场）。它更适合主观任务，对于事实准确性则不太可靠
- **Model-based evaluation**：使用判别模型和奖励模型来评估模型输出。它与人类偏好高度相关，但存在对其自身输出存在偏见以及评分不一致的问题。
- **Feedback signal**：分析错误模式以识别具体弱点，例如遵循复杂指令的限制、缺乏特定知识或易受对抗性提示的影响。这可以通过更好的数据生成和训练参数来改进。

> LLM评估的 学习资料

- [Evaluation guidebook](https://github.com/huggingface/evaluation-guidebook)：关于 LLM 评估的实用见解和理论知识。
- [Open LLM Leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)：主要排行榜，用于以开放和可重复的方式比较 LLMs（自动化基准测试）。
- [Language Model Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness)：一个使用自动化基准评估 LLMs 的流行框架。
- [Lighteval](https://github.com/huggingface/lighteval)：一个替代性评估框架，也包含基于模型的评估。
- [Chatbot Arena](https://lmarena.ai/) ：通用型 LLMs 的 Elo 评分，基于人类进行的比较（人工评估）。

### Quantization

量化是将模型的参数和激活使用较低精度进行转换的过程。例如，使用 16 位存储的权重可以转换为 4 位表示。这种技术在减少与 LLMs 相关的计算和内存成本方面变得越来越重要。

- 基础技术：学习不同的精度级别（FP32、FP16、INT8 等）以及如何使用 absmax 和零点技术进行朴素量化。
- GGUF & llama.cpp: 最初设计用于在 CPU 上运行，llama.cpp 和 GGUF 格式已成为在消费级硬件上运行 LLMs 的最流行工具。它支持将特殊标记、词汇和元数据存储在单个文件中。
- GPTQ & AWQ：GPTQ/EXL2 和 AWQ 等技术采用逐层校准方法，在极低位宽下仍能保持性能。它们通过动态缩放来减少灾难性异常值，选择性地跳过或重新校准权重最大的参数。
- SmoothQuant & ZeroQuant：新的量化友好转换（SmoothQuant）和基于编译器的优化（ZeroQuant）有助于在量化前减轻异常值。它们还通过融合某些操作和优化数据流来减少硬件开销。

> 量化的 学习资料

- [Introduction to quantization](https://mlabonne.github.io/blog/posts/Introduction_to_Weight_Quantization.html)：量化概述、absmax 和零点量化，以及 LLM.int8() 带代码。
- [Quantize Llama models with llama.cpp](https://mlabonne.github.io/blog/posts/Quantize_Llama_2_models_using_ggml.html)：教程介绍如何使用 llama.cpp 和 GGUF 格式量化 Llama 2 模型。
- [4-bit LLM Quantization with GPTQ](https://mlabonne.github.io/blog/posts/4_bit_Quantization_with_GPTQ.html) ：介绍如何使用 GPTQ 算法与 AutoGPTQ 量化 LLM。
- [Understanding Activation-Aware Weight Quantization](https://medium.com/friendliai/understanding-activation-aware-weight-quantization-awq-boosting-inference-serving-efficiency-in-10bb0faf63a8)：AWQ 技术的概述及其优势
- [SmoothQuant on Llama 2 7B](https://github.com/mit-han-lab/smoothquant/blob/main/examples/smoothquant_llama_demo.ipynb)：教程介绍如何在 8 位精度下使用 SmoothQuant 与 Llama 2 模型。
- [DeepSpeed Model Compression](https://www.deepspeed.ai/tutorials/model-compression/) ：教程介绍如何使用 ZeroQuant 和极限压缩（XTC）与 DeepSpeed 压缩技术。

### New Trends 

以下是一些不适合归入其他类别的显著主题。其中一些是成熟的技术（模型合并、多模态），但另一些则更具实验性（可解释性、测试时计算扩展），并且是众多研究论文的焦点。

- 模型合并：合并训练好的模型已成为一种无需微调即可创建高性能模型的热门方法。流行的 mergekit 库实现了最流行的合并方法，如 SLERP、DARE 和 TIES。
- 多模态模型：这些模型（如 CLIP、Stable Diffusion 或 LLaVA）通过统一的嵌入空间处理多种类型的输入（文本、图像、音频等），从而解锁了强大的应用，如文本到图像的转换。
- 可解释性：机制可解释性技术如稀疏自动编码器（SAEs）在提供关于 LLMs 内部工作机制的见解方面取得了显著进展。这也已应用于消融等技术，这些技术允许您在不训练模型的情况下修改模型的行为。
- 测试时计算：使用强化学习技术训练的推理模型可以通过在测试时扩展计算预算来进一步改进。这可能涉及多次调用、MCTS 或使用专门的模型，如过程奖励模型（PRM）。具有精确评分的迭代步骤可以显著提高复杂推理任务的表现。

> 对应的学习资料

- [Merge LLMs with mergekit](https://mlabonne.github.io/blog/posts/2024-01-08_Merge_LLMs_with_mergekit.html)：关于使用 mergekit 进行模型合并的教程。
- [Smol Vision](https://github.com/merveenoyan/smol-vision)：一个专注于小型多模态模型的笔记本和脚本集合。
- [Large Multimodal Models](https://huyenchip.com/2023/10/10/multimodal.html) ：多模态系统的概述以及该领域的近期历史。
- [Unsensor any LLM with abliteration](https://huggingface.co/blog/mlabonne/abliteration)：将可解释性技术直接应用于修改模型风格。
- [Intuitive Explanation of SAEs](https://adamkarvonen.github.io/machine_learning/2024/06/11/sae-intuitions.html) ：关于 SAEs 如何工作以及它们为何适用于可解释性的文章。
- [Scaling test-time compute](https://huggingface.co/spaces/HuggingFaceH4/blogpost-scaling-test-time-compute)：教程和实验，使用 3B 模型在 MATH-500 上超越 Llama 3.1 70B。

## LLM工程实践

![img](https://github.com/mlabonne/llm-course/raw/main/img/roadmap_engineer.png)

### Running LLMs

运行 LLMs 可能很困难，因为硬件要求很高。根据您的用例，您可能希望通过 API（如 GPT-4）简单地消费模型，或者在当地运行它。在任何情况下，额外的提示和指导技术都可以改进并约束您的应用程序的输出。

- LLM API：API 是部署 LLMs 的一种便捷方式。这个领域分为私有 LLMs（OpenAI、Google、Anthropic 等）和开源 LLMs（OpenRouter、Hugging Face、Together AI 等）。
- 开源 LLMs：Hugging Face Hub 是寻找 LLMs 的好地方。您可以直接在 Hugging Face Spaces 中运行其中的一些，或者下载并在 LM Studio 等应用程序中本地运行，或者通过命令行使用 llama.cpp 或 ollama 运行。
- 提示工程：常见技术包括零样本提示、少样本提示、思维链和 ReAct。它们在更大的模型上效果更好，但可以适应较小的模型。
- 结构化输出：许多任务需要结构化的输出，例如严格的模板或 JSON 格式。可以使用 Outlines 等库来指导生成并遵守给定的结构。一些 API 也原生支持使用 JSON 模式生成结构化输出。

> 参考资料

- [Run an LLM locally with LM Studio](https://www.kdnuggets.com/run-an-llm-locally-with-lm-studio) ： 关于如何使用 LM Studio 的简短指南。
- [Prompt engineering guide](https://www.promptingguide.ai/) ：包含示例的提示技术全面列表
- [Outlines - Quickstart](https://dottxt-ai.github.io/outlines/latest/quickstart/): Outlines 支持的引导生成技术列表。
- [LMQL - Overview](https://lmql.ai/docs/language/overview.html): LMQL 语言的介绍。

### Building a Vector Storage

创建向量存储是构建检索增强生成（RAG）管道的第一步。文档被加载、分割，并使用相关的片段来生成向量表示（嵌入），这些表示被存储以供推理时使用。

- 导入文档：文档加载器是方便的包装器，可以处理多种格式：PDF、JSON、HTML、Markdown 等。它们还可以直接从某些数据库和 API（GitHub、Reddit、Google Drive 等）中检索数据。
- 分割文档：文本分割器将文档分解为更小、具有语义意义的片段。与其在 n 个字符后分割文本，通常更好的方式是按标题或递归方式分割，并附带一些额外的元数据。
- 嵌入模型：嵌入模型将文本转换为向量表示。选择特定任务的模型可以显著提高语义搜索和 RAG 的性能。
- 向量数据库：向量数据库（如 Chroma、Pinecone、Milvus、FAISS、Annoy 等）旨在存储嵌入向量。它们能够基于向量相似性高效检索与查询“最相似”的数据。

> 参考资料

- [LangChain - Text splitters](https://python.langchain.com/docs/how_to/#text-splitters): LangChain 中实现的多种文本分割器列表。
- [Sentence Transformers library](https://www.sbert.net/)：用于嵌入模型的流行库。
- [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard): 嵌入模型的排行榜。
- [The Top 7 Vector Databases](https://www.datacamp.com/blog/the-top-5-vector-databases) ：对最佳和最受欢迎的向量数据库的比较。

### Retrieval Augmented Generation

通过 RAG，LLMs 从数据库中检索上下文文档以提高其回答的准确性。RAG 是一种在不进行任何微调的情况下增强模型知识的方法。

- **Orchestrators**：像 LangChain 和 LlamaIndex 这样的编排器是连接 LLMs 与工具和数据库的流行框架。模型上下文协议（MCP）引入了一个新标准，用于跨提供商向模型传递数据和上下文。
- **Retrievers**：查询重写器（如 CoRAG）和生成式检索器（如 HyDE）通过转换用户查询来增强搜索效果。多向量检索和混合检索方法结合嵌入和关键词信号，以提高召回率和精确率。
- **Memory**：为了记住之前的指令和答案，LLMs 和 ChatGPT 等聊天机器人会将这段历史添加到它们的上下文窗口中。这个缓冲区可以通过摘要（例如，使用一个较小的 LLM）、向量存储+RAG 等方式来改进。
- **Evaluation**：我们需要评估文档检索阶段（上下文精确度和召回率）以及生成阶段（忠实度和答案相关性）。这可以通过 Ragas 和 DeepEval 等工具简化（评估质量）。

> 参考资料

- [Llamaindex - High-level concepts](https://docs.llamaindex.ai/en/stable/getting_started/concepts.html): 构建 RAG 管道时需要了解的主要概念。
- [Model Context Protocol](https://modelcontextprotocol.io/introduction): 介绍 MCP 的动机、架构和快速入门。
- [Pinecone - Retrieval Augmentation](https://www.pinecone.io/learn/series/langchain/langchain-retrieval-augmentation/): 检索增强过程的概述。
- [LangChain - Q&A with RAG](https://python.langchain.com/docs/tutorials/rag/): 构建典型 RAG 流程的逐步教程。
- [LangChain - Memory types](https://python.langchain.com/docs/how_to/chatbots_memory/): 不同内存类型及其相关用途列表。
- [RAG pipeline - Metrics](https://docs.ragas.io/en/stable/concepts/metrics/index.html): 用于评估 RAG 流程的主要指标概述。

### Advanced RAG

实际应用可能需要复杂的流程，包括 SQL 或图数据库，以及自动选择相关工具和 API。这些高级技术可以改进基准解决方案并提供额外功能。

- 查询构建：传统数据库中存储的结构化数据需要特定的查询语言，如 SQL、Cypher、元数据等。我们可以通过查询构建直接将用户指令翻译成查询，以访问数据。
- 工具：代理通过自动选择最相关的工具来增强 LLMs，以提供答案。这些工具可以简单到使用 Google 或维基百科，也可以更复杂，如 Python 解释器或 Jira。
- 后处理：对输入到 LLM 的数据进行处理的最终步骤。通过重新排序、RAG 融合和分类，增强检索到的文档的相关性和多样性。
- 编程 LLMs：像 DSPy 这样的框架允许你以编程方式根据自动化评估来优化提示和权重。

> 参考资料

- [LangChain - Query Construction](https://blog.langchain.dev/query-construction/): 关于不同类型查询构建的博客文章。
- [LangChain - SQL](https://python.langchain.com/docs/tutorials/sql_qa/): 关于如何使用 LLMs 与 SQL 数据库交互的教程，涉及 Text-to-SQL 和可选的 SQL agent。
- [Pinecone - LLM agents](https://www.pinecone.io/learn/series/langchain/langchain-agents/): 介绍不同类型的agent和工具。
- [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) ：一篇更理论性的关于 LLM agent的文章。
- [LangChain - OpenAI's RAG](https://blog.langchain.dev/applying-openai-rag/): 概述 OpenAI 采用的 RAG 策略，包括后处理。
- [DSPy in 8 Steps](https://dspy-docs.vercel.app/docs/building-blocks/solving_your_task): 通用指南介绍 DSPy 的模块、签名和优化器。

### Agents

一个 LLM agent可以通过基于对环境的推理来执行任务，通常通过使用工具或函数与外部系统交互来实现自主操作。

- agent基础：agent通过思考（内部推理来决定下一步做什么）、行动（执行任务，通常通过交互外部工具）和观察（分析反馈或结果来完善下一步）来运行。
- agent框架：agent开发可以通过不同的框架来简化，例如 LangGraph（工作流的设计和可视化）、LlamaIndex（带有 RAG 的数据增强代理）或 smolagents（适合初学者的轻量级选项）。
- 多智能体：更多实验性框架包括不同智能体之间的协作，例如 CrewAI（基于角色的团队编排）、AutoGen（基于对话的多智能体系统）和 OpenAI Agents SDK（生产就绪且与强大的 OpenAI 模型深度集成）。

> 参考资料

- [Agents Course](https://huggingface.co/learn/agents-course/unit0/introduction): 由 Hugging Face 制作的关于 AI 智能体的热门课程。
- [AI Agents Comparison](https://langfuse.com/blog/2025-03-19-ai-agent-comparison) ：不同开源 AI 代理框架的功能比较。
- [LangGraph](https://langchain-ai.github.io/langgraph/concepts/why-langgraph/): 如何使用 LangGraph 构建 AI 代理的概述。
- [LlamaIndex Agents](https://docs.llamaindex.ai/en/stable/use_cases/agents/): 使用 LlamaIndex 构建代理的用例和资源。
- [smolagents](https://huggingface.co/docs/smolagents/index): 包含引导式游览、操作指南和更多概念性文章的文档。

### Inference optimization

文本生成是一个成本高昂的过程，需要昂贵的硬件。除了量化之外，还提出了各种技术来最大化吞吐量并降低推理成本。

- Flash Attention：优化注意力机制，将其复杂度从二次方降至线性，从而加速训练和推理。
- 键值缓存：理解键值缓存以及多查询注意力（MQA）和分组查询注意力（GQA）引入的改进。
- 推测解码：使用一个小模型来生成草稿，然后由一个更大的模型进行审阅，以加快文本生成速度。

> 参考资料

- [GPU Inference](https://huggingface.co/docs/transformers/main/en/perf_infer_gpu_one) ：解释如何优化 GPU 上的推理。
- [LLM Inference](https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices) ：如何优化生产环境中的 LLM 推理的最佳实践。
- [Optimizing LLMs for Speed and Memory](https://huggingface.co/docs/transformers/main/en/llm_tutorial_optimization) ：解释三种主要优化速度和内存的技术，即量化、Flash Attention 和架构创新。
- [Assisted Generation](https://huggingface.co/blog/assisted-generation) ：HF 的推测解码版本，这是一篇有趣的博客文章，介绍了它是如何与代码一起实现的工作原理。

### Deploying LLMs

大规模部署 LLMs 是一项工程壮举，可能需要多个 GPU 集群。在其他场景中，演示和本地应用程序可以通过更低的复杂度实现。

- 本地部署：隐私是开源 LLMs 相对于私有 LLMs 的一个重要优势。本地 LLM 服务器（如 LM Studio、Ollama、oobabooga、kobold.cpp 等）利用这一优势来支持本地应用程序。
- 演示部署：Gradio 和 Streamlit 等框架有助于原型设计应用程序和分享演示。您还可以轻松地在线托管它们，例如使用 Hugging Face Spaces。
- 服务器部署：大规模部署 LLMs 需要云（另见 SkyPilot）或本地基础设施，并通常利用优化的文本生成框架，如 TGI、vLLM 等。
- 边缘部署：在受限环境中，高性能框架如 MLC LLM 和 mnn-llm 可以在网络浏览器、Android 和 iOS 上部署 LLMs。

> 参考资料

- [Streamlit - Build a basic LLM app](https://docs.streamlit.io/knowledge-base/tutorials/build-conversational-apps): 使用 Streamlit 制作一个类似 ChatGPT 的基本应用的教程。
- [HF LLM Inference Container](https://huggingface.co/blog/sagemaker-huggingface-llm): 使用 Hugging Face 的推理容器在 Amazon SageMaker 上部署 LLMs。
- [Philschmid blog](https://www.philschmid.de/) ：关于使用 Amazon SageMaker 部署 LLM 的高质量文章集合。
- [Optimizing latence](https://hamel.dev/notes/llm/inference/03_inference.html) ：在吞吐量和延迟方面对 TGI、vLLM、CTranslate2 和 mlc 的比较。

### Securing LLMs

除了与软件相关的传统安全问题外，LLMs 由于其训练和提示方式，还存在独特的弱点。

- 提示词攻击：与提示词工程相关的不同技术，包括提示词注入（向模型答案中注入额外指令以劫持模型）、数据/提示词泄露（获取其原始数据/提示词）和越狱攻击（设计提示词以绕过安全功能）。
- 后门攻击：攻击向量可以针对训练数据本身，通过污染训练数据（例如，使用虚假信息）或创建后门（在推理期间改变模型行为的秘密触发器）来实现。
- 防御措施：保护你的 LLM 应用的最佳方式是测试它们针对这些漏洞（例如，使用红队测试和像 garak 这样的检查）并在生产环境中观察它们（使用像 langfuse 这样的框架）。

> 参考资料

- [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/) ：LLM 应用中最关键的 10 个漏洞列表。
- [Prompt Injection Primer](https://github.com/jthack/PIPE) ：为工程师准备的关于提示注入的简短指南。
- [LLM Security](https://llmsecurity.net/) by [@llm_sec](https://twitter.com/llm_sec): 与 LLM 安全相关的资源列表
- [Red teaming LLMs](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/red-teaming) by Microsoft: 关于如何使用 LLMs 进行 Red Teaming 的指南。





