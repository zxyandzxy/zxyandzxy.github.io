---
layout: post
title: "使用 LoRA 和 QLoRA 微调 LLM 的实验见解"
date: 2025-06-26
tags: [LLM]
comments: true
author: zxy
---

> 参考文章：[使用 LoRA 和 QLoRA 微调 LLMs：来自数百次实验的见解 - Lightning AI --- Finetuning LLMs with LoRA and QLoRA: Insights from Hundreds of Experiments - Lightning AI](https://lightning.ai/pages/community/lora-insights/)

## LoRA简介

LoRA，全称为 Low-Rank Adaptation ([Hu et al 2021](https://arxiv.org/abs/2106.09685))，在冻结原始模型参数的同时，向模型中添加少量可训练参数。LoRA 将一个权重矩阵分解为两个较小的权重矩阵，如下所示，以更参数高效的方式近似全监督微调。

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage7.png" alt="img" style="zoom: 50%;" />

## Evaluation Tasks and Dataset

本文的重点是选择最佳设置。为了保持合理的范围，数据集是固定，并仅专注于 LLMs 的监督指令微调。

对于模型评估，选择 Eleuther AI 评估工具箱中的一小部分任务，包括 TruthfulQA、BLiMP Causative、MMLU Global Facts，以及带有两位（arithmetic 2ds）和四位（arithmetic 4ds）的简单算术任务。

在每次基准测试中，模型性能得分在 0 到 1 之间进行归一化处理，其中 1 代表完美得分。TruthfulQA 报告两个得分，定义如下：

- MC1（单真）：给定一个问题以及 4-5 个答案选项，选择唯一正确的答案。模型的选取是它分配给问题后最高完成对数概率的答案选项，与其他答案选项无关。得分为所有问题的简单准确率。
- MC2（多真）：给定一个问题以及多个真/假参考答案，得分为分配给真答案集的归一化总概率。

作为参考，175B GPT-3 模型的 TruthfulQA MC1 和 MC2 值分别为 0.21 和 0.33。

以下是两个例子，用以说明算术 2D 和算术 4D 之间的区别：

- 算术 2D："59 减去 38 是多少？" "21"。
- 算术 4D："2762 加上 2751 是多少？" "5513"。

如上所述，数据集不变，使用广泛研究或更常用作监督指令微调的 Alpaca 数据集。当然，还有许多其他数据集可用于指令微调，包括 LIMA、Dolly、LongForm、FLAN 等。

Alpaca 数据集包含约 50k 个指令-响应对用于训练，输入大小的中位数为 110 个 token（使用 Llama 2 SentencePiece 分词器），如下面的直方图所示。

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage1.jpg" alt="img" style="zoom:50%;" />

数据集的任务本身可以按如下图所示的结构进行组织：

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage5.jpg" alt="img" style="zoom:50%;" />

## Code Framework

这篇文章的自定义 LLM 微调代码基于开源的 [Lit-GPT repository](https://github.com/Lightning-AI/lit-gpt) 仓库，具体逻辑如下：

（1）**Clone the repository and install the requirements**

```python
git clone https://github.com/Lightning-AI/lit-gpt
cd lit-gpt
pip install -r requirements.txt
```

（2）**Download and prepare a model checkpoint**

```python
python scripts/download.py --repo_id mistralai/Mistral-7B-Instruct-v0.1
# there are many other supported models
python scripts/convert_hf_checkpoint.py --checkpoint_dir checkpoints/mistralai/Mistral-7B-Instruct-v0.1
```

（3）**Prepare a dataset**

```python
python scripts/prepare_alpaca.py --checkpoint_dir checkpoints/mistralai/Mistral-7B-Instruct-v0.1
# or from a custom CSV file
python scripts/prepare_csv.py --csv_dir MyDataset.csv --checkpoint_dir checkpoints/mistralai/Mistral-7B-Instruct-v0.1
```

（4）**Finetune**

```python
python finetune/lora.py --checkpoint_dir checkpoints/mistralai/Mistral-7B-Instruct-v0.1/  --precision bf16-true
```

（5）**Merge LoRA weights**

```python
python scripts/merge_lora.py \
  --checkpoint_dir "checkpoints/mistralai/Mistral-7B-Instruct-v0.1" \
  --lora_path "out/lora/alpaca/Mistral-7B-Instruct-v0.1/lit_model_lora_finetuned.pth" \
  --out_dir "out/lora_merged/Mistral-7B-Instruct-v0.1/"

cp checkpoints/mistralai/Mistral-7B-Instruct-v0.1/*.json \
  out/lora_merged/Mistral-7B-Instruct-v0.1/
```

（6）**Evaluate**

```python
python eval/lm_eval_harness.py \
  --checkpoint_dir "out/lora_merged/Mistral-7B-Instruct-v0.1/" \
  --eval_tasks "[arithmetic_2ds, ..., truthfulqa_mc]" \
  --precision "bf16-true" \
  --batch_size 4 \
  --num_fewshot 0 \
  --save_filepath "results.json"
```

（7）**Use**

```python
python chat/base.py --checkpoint_dir "out/lora_merged/Mistral-7B-Instruct-v0.1/"
```

## Choosing a Good Base Model

第一个任务是为 LoRA 实验选择一个合适的基座模型。为此，需要专注于那些尚未进行指令微调的模型：[phi-1.5 1.3B](https://arxiv.org/abs/2309.05463), [Mistral 7B](https://arxiv.org/abs/2310.06825), [Llama 2 7B](https://arxiv.org/abs/2307.09288), Llama 2 13B, 和 [Falcon 40B](https://falconllm.tii.ae/).请注意，所有实验都在单个 A100 GPU 上运行。

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage2.jpg)

从上表可以看出，Mistral 7B 模型在数学基准测试中表现异常出色。与此同时，phi-1.5 1.3B 模型虽然规模相对较小，但在 TruthfulQA MC2 测试中展现了令人印象深刻的性能。不知何故，Llama 2 13B 在算术基准测试中表现不佳，而规模较小的 Llama 2 7B 在该领域则显著优于它。

由于研究人员和从业者目前推测 phi-1.5 1.3B 和 Mistral 7B 可能是在基准测试数据上训练的，因此在实验中未使用它们。此外，选择剩余模型中最小的模型可以在保持较低硬件需求的同时提供最大的改进空间。因此，本文的其余部分将专注于 Llama 2 7B。

## Evaluating the LoRA Defaults

首先，我使用以下默认设置评估了 LoRA 微调（这些设置可以在 [finetune/lora.py](https://github.com/Lightning-AI/lit-gpt/blob/bf60124fa72a56436c7d4fecc093c7fc48e84433/finetune/lora.py#L38) 脚本中更改）

```python
# Hyperparameters
learning_rate = 3e-4
batch_size = 128
micro_batch_size = 1
max_iters = 50000  # train dataset size
weight_decay = 0.01
lora_r = 8
lora_alpha = 16
lora_dropout = 0.05
lora_query = True
lora_key = False
lora_value = True
lora_projection = False
lora_mlp = False
lora_head = False
warmup_steps = 100
```

（请注意，批大小为 128，但我们使用微批大小为 1 的梯度累积来节省内存；这会导致与常规批大小为 128 的训练相同的训练轨迹。如果对梯度累积的工作原理感到好奇，请查看文章《 [Finetuning LLMs on a Single GPU Using Gradient Accumulation](https://lightning.ai/blog/gradient-accumulation/)》）

该配置训练了 4,194,304 个 LoRA 参数，总共有 6,738,415,616 个可训练参数，在机器上使用单个 A100 大约花费了 1.8 小时。最大内存使用量为 21.33 GB。

为了衡量差异，重复进行了三次实验，以观察运行之间性能的波动情况。

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage12.jpg)

如上图所示，运行间的性能非常一致且稳定。值得注意的是，LoRA 默认模型的算术能力变得很差，但这或许可以预料，因为据我所知，Alpaca 不包含（太多）算术任务。

此外，还研究了 Meta 使用 RLHF 指令微调的 7B Llama 2 版本。根据下表可见，Meta 的 Llama 2 Chat 模型的算术性能也较差。然而，Chat 模型在其他基准测试（除 BLiMP 外）上有了很大改进，这可以作为我们希望用 LoRA 微调来接近的目标参考。

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage10.jpg)

## Memory Savings with QLoRA

在开始调整 LoRA 超参数之前，我们探讨一下 QLoRA（由 Dettmers 等人开发的热门量化 LoRA 技术）在建模性能和内存节省之间的权衡。我们可以通过在 Lit-GPT 中使用–quantize 标志（此处为 4 位 Normal Float 类型）来启用 QLoRA，具体如下：

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage9.jpg" alt="img" style="zoom: 50%;" />

此外，我还尝试了 4 位浮点数精度作为对照。以下是其对训练时间和最大内存使用的影响：

Default LoRA (with bfloat-16):

- Training time: 6685.75s
- Memory used: 21.33 GB

QLoRA via –-quantize “bnb.nf4”:

- Training time: 10059.53s
- Memory used: 14.18 GB

QLoRA via –quantize “bnb.fp4”:

- Training time: 9334.45s
- Memory used: 14.19 GB

我们可以看到，QLoRA 将内存需求减少了近 6 GB。然而，代价是训练时间变慢了 30%，这是由于额外的量化和反量化步骤所导致的，符合预期。接下来，让我们看看 QLoRA 训练如何影响模型性能：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage3.jpg)

从上表可以看出，与常规的 LoRA 相比，QLoRA 确实对模型性能产生了一定影响。模型在算术基准测试中有所提升，但在 MMLU Global Facts 基准测试中表现下降。由于内存节省非常显著（这通常可以抵消更长的训练时间，因为它允许用户在更小的 GPU 上运行模型），将在本文的其余部分使用 QLoRA。

## Learning Rate Schedulers and SGD

在所有之前的实验中都使用了 AdamW 优化器，因为它是 LLM 训练的常见选择。然而，众所周知，Adam 优化器可能非常内存密集。这是因为它为每个模型参数引入并跟踪两个额外的参数（动量 m 和 v）。大型语言模型（LLMs）具有许多模型参数；例如，我们的 Llama 2 模型有 70 亿个模型参数。

本节探讨了是否值得用 SGD 优化器替换 AdamW。然而，对于 SGD 优化器来说，引入学习率调度器尤为重要。我选择了余弦退火调度器，它在每次批次更新后降低学习率。

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage11.jpg" alt="img" style="zoom:33%;" />

不幸的是，将 AdamW 替换为 SGD 仅带来了微小的内存节省。（AdamW: 14.18 GB， SGD: 14.15 GB）

这可能是由于大部分内存用于大型矩阵乘法，而不是将额外参数保留在内存中。

但这个微小差异或许在预料之中。在当前选择的 LoRA 配置（r=8）下，我们有 4,194,304 个可训练参数。如果 Adam 为每个模型参数增加 2 个额外值，这些值以 16 位浮点数存储，那么我们有 4,194,304 * 2 * 16 bit = 134.22 Mbit = 16.78 MB。

当我们增加 LoRA 的 r 值到 256 时，我们可以观察到更大的差异，我们稍后会这样做。在 r=256 的情况下，我们有 648,871,936 个可训练参数，使用上述相同计算方法，这相当于 2.6 GB。实际测量结果显示差异为 3.4 GB，这可能是由于存储和复制优化器状态时存在一些额外开销。

归根结底，对于可训练参数较少的情况，例如 LoRA 和低 r（秩）值的情况，用 SGD 替换 AdamW 带来的内存收益可能非常小，这与预训练时我们训练更多参数的情况形成对比。

尽管 SGD 在这里没有为我们提供显著的内存节省，但让我们还是快速看一下结果模型的性能：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage13.jpg)

似乎 SGD 优化器的性能与 AdamW 相当。有趣的是，当给 AdamW 添加调度器时，TruthfulQA MC2 和 MMLU Global Facts 的性能有所提升，但算术性能有所下降。（注：TruthfulQA MC2 是一个被广泛认可的基准，也出现在其他公共排行榜中。）目前，我们不会过分强调算术性能，并将使用带有调度器的 AdamW 继续进行本文的其余实验。

如果你想要复现这些实验，我发现最佳的 AdamW 学习率是 3e-4，衰减率为 0.01。最佳的 SGD 学习率是 0.1，动量为 0.9。在这两种情况下，我都额外使用了 100 步的学习率预热。（基于这些实验，余弦调度器已被添加到 [Lit-GPT](https://github.com/Lightning-AI/lit-gpt/pull/626) 中，现在默认启用。）

## Iterating Over the Dataset Multiple Times

到目前为止，我已经用 50k 次迭代训练了所有模型——Alpaca 数据集有 50k 个训练样本。一个明显的问题是，我们是否可以通过多次迭代训练集来提高模型性能，因此我运行了之前的实验，使用了 100k 次迭代，这是一个 2 倍的提升：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage6.jpg)

有趣的是，增加迭代次数会导致整体性能变差。这种下降在算术基准测试中最为显著。我的假设是，Alpaca 数据集不包含任何相关的算术任务，当模型更专注于其他任务时，它会主动忘记基本的算术知识。

## LoRA Hyperparameter Tuning Part 1: LoRA for All Layers

现在我们已经探索了围绕 LoRA 微调脚本的基本设置，让我们将注意力转向 LoRA 超参数本身。默认情况下，LoRA 仅对多头自注意力块中的键和查询矩阵启用。现在，我们也将它对值矩阵、投影层和线性层启用：

<img src="https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage18.jpg" alt="img" style="zoom:25%;" />

## LoRA Hyperparameter Tuning Part 2: Increasing R

LoRA 参数中最重要的一个参数是“r”，它决定了 LoRA 矩阵的秩或维度，直接影响模型的复杂度和容量。较高的“r”意味着更强的表达能力，但可能导致过拟合，而较低的“r”则可以通过牺牲表达能力来减少过拟合。让我们对所有层都启用 LoRA，并将 r 从 8 增加到 16，看看这对性能有什么影响：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage17.jpg)

我们可以看到，仅仅增加 r 本身就让结果变差了，那么发生了什么呢？让我们在下一节中找出答案。

## LoRA Hyperparameter Tuning Part 3: Changing Alpha

在上一节中，我们增加了矩阵的秩 r，而 LoRA 的 alpha 参数保持不变。较高的“alpha”会更多地强调低秩结构或正则化，而较低的“alpha”会减少其影响，使模型更多地依赖原始参数。调整“alpha”有助于在拟合数据和通过正则化防止过拟合之间取得平衡。

通常情况下，微调 LLMs 时选择 alpha 的大小通常是秩的两倍（注意：这与使用扩散模型时不同）。让我们试试看当我们把 alpha 增加一倍时会发生什么：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage8.jpg)

正如我们所见，将 alpha 增加到 32 现在得到了我们迄今为止最好的模型！但我们再次用更多的待训练参数代价换来了这个改进：

r=8:

- Number of trainable parameters: 20,277,248
- Number of non trainable parameters: 6,738,415,616
- Memory used: 16.42 GB

r=16:

- Number of trainable parameters: 40,554,496
- Number of non trainable parameters: 6,738,415,616
- Memory used: 16.47 GB

然而，可训练参数的数量仍然足够小，以至于不会明显影响峰值内存需求。无论如何，我们现在终于开始取得一些进展，并通过更明显的幅度提升模型性能。所以，让我们继续前进，看看通过增加秩和 alpha，我们能把模型推向多远：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage4.jpg)

我还进行了使用极大秩（512、1024 和 2048）的额外实验，但这些实验的结果更差。有些实验甚至在训练过程中都没有收敛到接近零的损失，因此我没有将它们添加到表格中。

到目前为止，我们可以注意到最后一行中 r=256 和 alpha=512 的模型在整体性能上表现最佳。作为一个额外的控制实验，我用 alpha=1 重复了运行，并注意到确实需要一个较大的 alpha 值才能获得良好的性能：

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage14.jpg)

我还用 16 和 32 的 alpha 值重复了实验，并且观察到与将 alpha 值设为秩的两倍相比，性能同样更差。

## LoRA Hyperparameter Tuning Part 3: Very Large R

对于本文的最终调优实验，我想进一步优化上一节最佳模型（r=256，最后一行）的 alpha 值，怀疑它可能有点太大。

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/lora-expimage15.jpg)

如上表所示，当增加秩时，选择较大的 alpha 值似乎至关重要。

对于 r=256 和 a=512 的 QLoRA 模型，很明显我们的模型在基准模型上取得了显著改进。微调后的模型仅在 4 位算术方面表现不如基准模型。然而，考虑到 Alpaca 数据集可能不包含此类训练示例，这一点是可以理解的。

如上所述，选择 alpha 为秩的两倍（例如 r=256 和 alpha=512）的常见建议确实产生了最佳结果，而较小的 alpha 值则导致更差的结果。那么，如果将 alpha 增加到“两倍秩”的建议之上会怎样呢？

![img](https://lightningaidev.wpengine.com/wp-content/uploads/2023/10/loraexp-2fold-rank.png)

根据上表中提供的结果，选择超过“两倍秩”建议的 alpha 值也会使基准测试结果变差。

## Conclusion

本文探讨了在使用 LoRA 训练自定义 LLMs 时可以调整的各种参数。我们发现，尽管 QLoRA 会带来运行时成本的增加，但它是一种很好的内存节省方法。此外，虽然学习率调度器可能有益，但在 AdamW 和 SGD 优化器之间选择几乎没有区别。而且，多次迭代数据集会使结果更糟。通过优化 LoRA 设置（包括秩）可以实现最佳性价比。增加秩会导致更多可训练参数，这可能导致更高的过拟合程度和运行时成本。然而，在增加秩时，选择合适的 alpha 值非常重要。