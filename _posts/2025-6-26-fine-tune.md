---
layout: post
title: "微调LLM"
date: 2025-06-26
tags: [LLM]
comments: true
author: zxy
---

# Fine-tune Llama 3.1 Ultra-Efficiently with Unsloth

> 参考文章：[使用 Unsloth 以超高效的方式微调 Llama 3.1 --- Fine-tune Llama 3.1 Ultra-Efficiently with Unsloth](https://huggingface.co/blog/mlabonne/sft-llama3)

Llama 3.1 的最新发布提供了具有惊人性能的模型，缩小了闭源模型和开源权重模型之间的差距。与其使用参数固定的通用 LLM 模型，如 GPT-4o 和 Claude 3.5，我们可以针对特定用例微调 Llama 3.1，以在更低的成本下实现更好的性能和可定制性。

![img](https://i.imgur.com/u0rJPa6.png)

本文全面概述监督微调，并与提示工程进行比较，以了解何时使用监督微调比较合适，详细说明主要技术及其优缺点，并介绍主要概念，如 LoRA 超参数、存储格式和聊天模板。最后，通过在 Google Colab 中使用 Unsloth 进行最先进的优化来实际实现微调 Llama 3.1 8B。

本文中使用的所有代码都可以在 [Google Colab](https://colab.research.google.com/drive/164cg_O7SV7G8kZr_JXqLd6VC7pd86-1Z#scrollTo=PoPKQjga6obN) 和  [LLM Course ](https://github.com/mlabonne/llm-course)中找到。

## Supervised Fine-Tuning

![img](https://i.imgur.com/0akg8cN.png)

监督微调（SFT）是一种改进和定制预训练 LLMs 的方法。它涉及在更小的指令和答案数据集上重新训练基础模型。主要目标是将一个预测文本的基本模型转变为能够遵循指令和回答问题的助手。SFT 还可以提高模型的整体性能，添加新知识，或将其适应特定任务和领域。微调后的模型可以经过一个可选的偏好对齐阶段（见 [DPO](https://mlabonne.github.io/blog/posts/Fine_tune_Mistral_7b_with_DPO.html)）以去除不需要的响应，修改其风格等。

下图展示了一个指令样本。它包括一个用于引导模型的系统提示、一个用于提供任务的用户提示以及模型预期生成的输出。

![img](https://i.imgur.com/RqlJEtH.png)

在考虑 SFT 之前，我建议尝试提示工程技术，如少样本提示或检索增强生成（RAG）。在实践中，这些方法可以在无需微调的情况下解决许多问题，使用闭源或开源权重模型（例如 Llama 3.1 Instruct）。如果这种方法无法满足您的目标（在质量、成本、延迟等方面），那么当指令数据可用时，SFT 成为一个可行的选项。请注意，SFT 还提供额外控制和可定制性的好处，以创建个性化的 LLMs。

然而，SFT 也有局限性。它最适合利用基础模型中已有的知识。如果学习完全新的信息，如未知语言，可能具有挑战性并导致更频繁的幻觉。对于基础模型不了解的新领域，建议首先在原始数据集上持续预训练它。

## SFT Techniques

三种最流行的 SFT 技术是全量微调、LoRA 和 QLoRA。

![img](https://i.imgur.com/P6sLsxl.png)

全量微调是最直接的超大规模语言模型微调技术。它涉及在指令数据集上重新训练预训练模型的所有参数。这种方法通常能提供最佳结果，但需要大量的计算资源（微调一个 8B 模型需要多个高端 GPU）。由于它会修改整个模型，因此它也是最具破坏性的方法，可能导致先前技能和知识的灾难性遗忘。

低秩适配（**Low-Rank Adaptation** LoRA）是一种流行的参数高效微调技术。它不会重新训练整个模型，而是冻结权重，并在每个目标层引入小型适配器（低秩矩阵）。这使得 LoRA 训练的参数数量远低于完整微调（不到 1%），从而减少了内存使用和训练时间。由于原始参数被冻结，这种方法是非破坏性的，因此适配器可以随意切换或组合。

QLoRA（**Quantization-aware Low-Rank Adaptation** 量化感知低秩适配）是 LoRA 的扩展，提供了更高的内存节省。与标准 LoRA 相比，它最多可减少 33%的内存使用，在 GPU 内存受限时特别有用。这种效率的提升是以更长的训练时间为代价的，QLoRA 的训练时间通常比常规 LoRA 多约 39%。

虽然 QLoRA 需要更长的训练时间，但其显著的内存节省使其成为 GPU 内存有限场景中唯一可行的选项。因此，在下一节中，将使用这种技术来在 Google Colab 上微调一个 Llama 3.1 8B 模型。

## Fine-Tune Llama 3.1 8B

要高效地微调[Llama 3.1 8B](https://huggingface.co/meta-llama/Meta-Llama-3.1-8B) 模型，我们将使用 Daniel 和 Michael Han 开发的 [Unsloth](https://github.com/unslothai/unsloth) 库。得益于其自定义内核，Unsloth 提供的训练速度比其他选项快 2 倍，内存使用量减少 60%，使其在 Colab 等受限环境中非常理想。遗憾的是，目前 Unsloth 仅支持单 GPU 设置。对于多 GPU 设置，我推荐使用  [TRL](https://huggingface.co/docs/trl/en/index)  和 [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) 等流行替代方案（两者也都将 Unsloth 作为后端）。

在这个例子中，我们将使用  [mlabonne/FineTome-100k](https://huggingface.co/datasets/mlabonne/FineTome-100k) 数据集对其进行 QLoRA 微调。这是 [arcee-ai/The-Tome](https://huggingface.co/datasets/arcee-ai/The-Tome) 的一个子集（不包含  [arcee-ai/qwen2-72b-magpie-en](https://huggingface.co/datasets/arcee-ai/qwen2-72b-magpie-en)），我使用 [HuggingFaceFW/fineweb-edu-classifier](https://huggingface.co/HuggingFaceFW/fineweb-edu-classifier) 重新进行了筛选。请注意，这个分类器并非为指令数据质量评估而设计，但我们可以将其用作一个粗略的近似值。生成的 FineTome 是一个超高质量的数据集，包含对话、推理问题、函数调用等内容。

安装所有必备的库

```python
!pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
!pip install --no-deps "xformers<0.0.27" "trl<0.9.0" peft accelerate bitsandbytes
```

安装完成后，我们可以按以下方式导入它们。

```python
import torch
from trl import SFTTrainer
from datasets import load_dataset
from transformers import TrainingArguments, TextStreamer
from unsloth.chat_templates import get_chat_template
from unsloth import FastLanguageModel, is_bfloat16_supported
```

现在我们来加载模型。由于我们想使用 QLoRA，我选择了预量化的 [unsloth/Meta-Llama-3.1-8B-bnb-4bit](https://huggingface.co/unsloth/Meta-Llama-3.1-8B-bnb-4bit)。这个 4 位精度的  [meta-llama/Meta-Llama-3.1-8B](https://huggingface.co/blog/mlabonne/meta-llama/Meta-Llama-3.1-8B) 版本比原始的 16 位精度模型（16GB）要小得多（5.4GB），下载速度也更快。我们使用 bitsandbytes 库以 NF4 格式加载。

在加载模型时，我们必须指定一个最大序列长度，这将限制其上下文窗口。Llama 3.1 支持高达 128k 的上下文长度，但在这个示例中我们将它设置为 2,048，因为这样会消耗更多的计算资源和 VRAM。最后， `dtype` 参数会自动检测你的 GPU 是否支持 BF16 格式，以在训练过程中提供更高的稳定性（此功能仅限于 Ampere 及更新的 GPU）。

```python
max_seq_length = 2048
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Meta-Llama-3.1-8B-bnb-4bit",
    max_seq_length=max_seq_length,
    load_in_4bit=True,
    dtype=None,
)
```

现在我们的模型以 4 位精度加载完毕，我们希望使用 LoRA 适配器进行参数高效的微调。LoRA 有三个重要的参数：

- 决定 LoRA 矩阵大小的秩（r）。秩通常从 8 开始，但可以高达 256。更高的秩可以存储更多信息，但会增加 LoRA 的计算和内存成本。在这里我们将其设置为 16。
- Alpha (α)，一个用于更新的缩放因子。Alpha 直接影响到适配器的贡献，通常设置为秩值的 1 倍或 2 倍。
- 目标模块：LoRA 可以应用于各种模型组件，包括注意力机制（Q、K、V 矩阵）、输出投影、前馈块和线性输出层。虽然最初专注于注意力机制，但将 LoRA 扩展到其他组件显示出其优势。然而，适应更多模块会增加可训练参数的数量和内存需求。

这里我们设置 r=16，α=16，并使每个线性模块都最大化质量。我们不使用 dropout 和偏差以加快训练。

此外，我们将使用 [Rank-Stabilized LoRA](https://arxiv.org/abs/2312.03732)（rsLoRA），该技术将 LoRA 适配器的缩放因子调整为与 1/√r 成正比，而不是与 1/r 成正比。这有助于稳定学习过程（尤其是对于更高适配器秩的情况），并随着秩的增加提升微调性能。梯度检查点由 Unsloth 处理，将输入和输出嵌入卸载到磁盘并节省 VRAM（显存）。

```python
model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    lora_alpha=16,
    lora_dropout=0,
    target_modules=["q_proj", "k_proj", "v_proj", "up_proj", "down_proj", "o_proj", "gate_proj"], 
    use_rslora=True,
    use_gradient_checkpointing="unsloth"
)
```

通过这个 LoRA 配置，我们只会训练 8 亿参数中的 4200 万（0.5196%）。这显示了 LoRA 与完整微调相比，效率要高多少。

现在让我们加载和准备我们的数据集。指令数据集以特定格式存储：可以是 Alpaca、ShareGPT、OpenAI 等。首先，我们想要解析这种格式以获取我们的指令和答案。我们的  [mlabonne/FineTome-100k](https://huggingface.co/datasets/mlabonne/FineTome-100k) 数据集使用 ShareGPT 格式，其中包含一个独特的 "conversations" 列，该列包含 JSONL 格式的消息。与 Alpaca 等更简单的格式不同，ShareGPT 非常适合存储多轮对话，这更接近用户与 LLMs 互动的方式。

一旦我们的指令-答案对被解析后，我们希望将它们重新格式化为遵循聊天模板。聊天模板是用户和模型之间进行对话的一种结构方式。它们通常包含特殊标记来识别消息的开始和结束、谁在说话等。基础模型没有聊天模板，因此我们可以选择任何一种：ChatML、Llama3、Mistral 等。在开源社区中，ChatML 模板（最初来自 OpenAI）是一个流行的选项。它简单地添加了两个特殊标记（ `<|im_start|>` 和 `<|im_end|>` ）来表示谁在说话。

如果我们把这个模板应用到之前的指令样本上，我们会得到以下结果：

```
<|im_start|>system
You are a helpful assistant, who always provide explanation. Think like you are answering to a five year old.<|im_end|>
<|im_start|>user
Remove the spaces from the following sentence: It prevents users to suspect that there are some hidden products installed on theirs device.
<|im_end|>
<|im_start|>assistant
Itpreventsuserstosuspectthattherearesomehiddenproductsinstalledontheirsdevice.<|im_end|>
```

在以下代码块中，我们使用 `mapping` 参数解析 ShareGPT 数据集，并包含 ChatML 模板。然后我们加载并处理整个数据集，将聊天模板应用于每场对话。

现在我们可以指定我们运行的训练参数了。下面简要介绍最重要的超参数：

- **Learning rate**: 它控制模型更新参数的强度。太低会导致训练缓慢，并可能陷入局部最小值。太高则可能导致训练不稳定或发散，从而降低性能。
- **LR scheduler**: 它在训练过程中调整学习率，开始时使用较高的 LR 以实现快速初始进展，然后在后期阶段逐渐降低。线性调度器和余弦调度器是最常见的两种选项。
- **Batch size**: 在权重更新之前处理的样本数量。较大的批量大小通常能带来更稳定的梯度估计，并可能提高训练速度，但它们也需要更多的内存。梯度累积通过在多次前向/后向传递后累积梯度，从而允许有效地使用更大的批量大小来更新模型。
- **Num epochs**: 指完整遍历训练数据集的次数。更多的周期可以让模型多次看到数据，有可能带来更好的性能。然而，过多的周期可能导致过拟合。
- **Optimizer**: 于调整模型参数以最小化损失函数的算法。在实践中，强烈推荐使用 AdamW 8 位：它的性能与 32 位版本相当，同时使用的 GPU 内存更少。AdamW 的分页版本仅在分布式设置中才有意义。
- **Weight decay**: 一种正则化技术，通过在损失函数中添加对大权重的惩罚来实现。它通过鼓励模型学习更简单、更具泛化能力的特征来帮助防止过拟合。然而，过多的权重衰减可能会阻碍学习。
- **Warmup steps**: 训练开始时的一段时间，将学习率从较小的值逐渐增加到初始学习率。预热可以通过让模型在做出大幅更新之前适应数据分布，从而帮助稳定早期训练，尤其是在使用较大的学习率或批处理大小的情况下。
- **Packing**: 批次具有预定义的序列长度。我们不必为每个样本分配一个批次，而是可以将多个小样本组合在一个批次中，从而提高效率。

在 Google Colab 上使用 A100 GPU（40 GB 显存）在整个数据集（10 万样本）上训练了模型。训练耗时 4 小时 45 分钟。当然，你也可以使用显存更小的 GPU 和更小的批量大小，但速度远不如前者。例如，在 L4 上大约需要 19 小时 40 分钟，而在免费的 T4 上则高达 47 小时。

在这种情况下，我建议只加载数据集的一部分以加快训练速度。你可以通过修改之前的代码块，比如 `dataset = load_dataset("mlabonne/FineTome-100k", split="train[:10000]")` ，只加载 10k 样本。或者，你也可以使用 Paperspace、RunPod 或 Lambda Labs 等更便宜的云 GPU 服务商。

```python
trainer=SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=max_seq_length,
    dataset_num_proc=2,
    packing=True,
    args=TrainingArguments(
        learning_rate=3e-4,
        lr_scheduler_type="linear",
        per_device_train_batch_size=8,
        gradient_accumulation_steps=2,
        num_train_epochs=1,
        fp16=not is_bfloat16_supported(),
        bf16=is_bfloat16_supported(),
        logging_steps=1,
        optim="adamw_8bit",
        weight_decay=0.01,
        warmup_steps=10,
        output_dir="output",
        seed=0,
    ),
)

trainer.train()
```

模型训练完成后，让我们用一个简单的提示来测试它。这不是一个严格的评估，只是一个快速检查以发现潜在问题。我们使用 `FastLanguageModel.for_inference()` 来获得 2 倍的推理速度。

```python
model = FastLanguageModel.for_inference(model)

messages = [
    {"from": "human", "value": "Is 9.11 larger than 9.9?"},
]
inputs = tokenizer.apply_chat_template(
    messages,
    tokenize=True,
    add_generation_prompt=True,
    return_tensors="pt",
).to("cuda")

text_streamer = TextStreamer(tokenizer)
_ = model.generate(input_ids=inputs, streamer=text_streamer, max_new_tokens=128, use_cache=True)

```

模型的回应是"9.9"，这是正确的！（目前还没有进行偏好对齐，所以没有回答 No 这种更直观上理解的答案）

现在我们来保存我们训练好的模型。如果你记得 LoRA 和 QLoRA 的部分，我们训练的不是模型本身，而是一组适配器。Unsloth 中有三种保存方法： `lora` 仅保存适配器， `merged_16bit` / `merged_4bit` 将适配器与模型在 16 位/ 4 位精度下合并。

下面我们将它们以 16 位精度合并以最大化质量。我们首先在"model"目录本地保存，然后上传到 Hugging Face Hub。你可以在 [mlabonne/FineLlama-3.1-8B](https://huggingface.co/mlabonne/FineLlama-3.1-8B) 上找到训练好的模型。

```python
model.save_pretrained_merged("model", tokenizer, save_method="merged_16bit")
model.push_to_hub_merged("mlabonne/FineLlama-3.1-8B", tokenizer, save_method="merged_16bit")
```

Unsloth 还允许你直接将模型转换为 GGUF 格式。这是一种为 llama.cpp 创建的量化格式，与大多数推理引擎兼容，例如 [LM Studio](https://lmstudio.ai/), [Ollama](https://ollama.com/), 和 oobabooga 的 [text-generation-webui](https://github.com/oobabooga/text-generation-webui)。由于你可以指定不同的精度（ [article about GGUF and llama.cpp](https://mlabonne.github.io/blog/posts/Quantize_Llama_2_models_using_ggml.html)），我们将遍历一个列表，在 `q2_k` 、 `q3_k_m` 、 `q4_k_m` 、 `q5_k_m` 、 `q6_k` 、 `q8_0` 中进行量化，并将这些量化文件上传到 Hugging Face。 [mlabonne/FineLlama-3.1-8B-GGUF](https://huggingface.co/mlabonne/FineLlama-3.1-8B-GGUF) 包含了我们所有的 GGUF 文件。

```python
quant_methods = ["q2_k", "q3_k_m", "q4_k_m", "q5_k_m", "q6_k", "q8_0"]
for quant in quant_methods:
    model.push_to_hub_gguf("mlabonne/FineLlama-3.1-8B-GGUF", tokenizer, quant)
```

我们从头开始微调了一个模型，并上传了你可以现在在最喜欢的推理引擎中使用的量化文件。欢迎尝试 [mlabonne/FineLlama-3.1-8B-GGUF](https://huggingface.co/mlabonne/FineLlama-3.1-8B-GGUF) 上的最终模型。接下来该做什么？这里有一些关于如何使用你模型的建议：

- 在  [Open LLM Leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)上评估它（你可以免费提交）或使用其他评估方式，如 [LLM AutoEval ](https://github.com/mlabonne/llm-autoeval)中的评估。
- 使用偏好数据集如 [mlabonne/orpo-dpo-mix-40k](https://huggingface.co/datasets/mlabonne/orpo-dpo-mix-40k)  与直接偏好优化对齐，以提升性能
- 使用 [AutoQuant](https://colab.research.google.com/drive/1b6nqC7UZVt8bx4MksX7s656GXPM-eWw4?usp=sharing). 将其量化为 EXL2、AWQ、GPTQ 或 HQQ 等其他格式，以实现更快的推理或更低精度。
- 在 Hugging Face Space 上使用  [ZeroChat](https://colab.research.google.com/drive/1LcVUW5wsJTO2NGmozjji5CkC--646LgC) 部署它，适用于已经充分训练到可以遵循聊天模板的模型（约 20k 样本）。

# Axolotl - Documentation

> 参考文章：[Axolotl](https://docs.axolotl.ai/)

Axolotl 是一个旨在简化各种 AI 模型 post-training 的工具，它具备以下特性：

- 多模型支持：可训练 LLaMA、Mistral、Mixtral、Pythia 等多种模型。我们兼容 HuggingFace transformers 的 causal language models。
- 训练方法：全量微调、LoRA、QLoRA、GPTQ、QAT、偏好微调（DPO、IPO、KTO、ORPO）、强化学习（GRPO）、多模态、奖励建模（RM）/过程奖励建模（PRM）。
- 易于配置：可在数据集预处理、训练、评估、量化、推理等环节复用单个 YAML 文件。
- 性能优化：多包处理、闪存注意力机制、Xformers、灵活注意力机制、Liger 内核、交叉熵剪裁、序列并行（SP）、LoRA 优化、多 GPU 训练（FSDP1、FSDP2、DeepSpeed）、多节点训练（Torchrun、Ray），以及更多
- 灵活的数据集处理：可从本地、HuggingFace 和云（S3、Azure、GCP、OCI）数据集加载数据。
- 云平台适配：我们提供 Docker 镜像，同时也提供 PyPI 包，适用于云平台和本地硬件。

## Installation 

> 配置需求

- NVIDIA GPU / AMD GPU
- Python ≥3.11
- PyTorch ≥2.5.1

> Installation Methods

在本地环境中安装 Axolotl 之前，请确保已安装 Pytorch，然后通过PyPI 安装

```python
pip3 install -U packaging setuptools wheel ninja
pip3 install --no-build-isolation axolotl[flash-attn,deepspeed]
```

我们使用 `--no-build-isolation` 来检测已安装的 PyTorch 版本（如果 安装时) 以免覆盖它，并确保我们设置正确的版本 依赖项，这些依赖项是特定于 PyTorch 版本或其他已安装 共依赖项。

## Quickstart

> 从使用 LoRA 微调一个小型语言模型开始。这个例子使用一个 1B 参数的模型，以确保它能在大多数 GPU 上运行。

```python
axolotl fetch examples   # Download example configs
axolotl train examples/llama-3/lora-1b.yml # Run the training
```

>The  Configuration  File

YAML 配置文件控制你训练的各个方面。以下是我们示例配置文件（部分）的样子

```python
base_model: NousResearch/Llama-3.2-1B

load_in_8bit: true
adapter: lora

datasets:
  - path: teknium/GPT4-LLM-Cleaned
    type: alpaca
dataset_prepared_path: last_run_prepared
val_set_size: 0.1
output_dir: ./outputs/lora-out
```

- `load_in_8bit: true` 和 `adapter: lora` 启用 LoRA 适配器微调。
- 为了进行完整微调，请删除这两行。
- 为了进行 QLoRA 微调，请替换为 `load_in_4bit: true` 和 `adapter: qlora` 

>Training

当运行 `axolotl train` 时，Axolotl：

- 下载基础模型
- (如果指定) 应用 QLoRA/LoRA 适配器层
- 加载并处理数据集
- 运行训练循环
- 保存训练好的模型和 / 或 LoRA 权重

# 其他的一些微调工具

> 参考文章：[大模型训练工具，小白也能轻松搞定！ - Milkha - 博客园](https://www.cnblogs.com/gzyatcnblogs/p/18684805)

<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003824646-1221341470.png" alt="img" style="zoom:50%;" />



## [Axolotl](https://github.com/axolotl-ai-cloud/axolotl)

[![img](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003851039-758908601.png)](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003851039-758908601.png)

Axolotl 是一款旨在简化各种人工智能模型微调的工具，支持多种配置和架构。

**主要特点：**

- 支持的常见开源大模型，多种训练方式，包括：全参微调、LoRA/QLoRA、xformers等。
- 可通过 yaml 或 CLI 自定义配置。
- 支持多种数据集格式以及自定义格式。
- 集成了 xformer、flash attention、liger kernel、rope 及 multipacking。
- 使用 Docker 在本地或云端轻松运行。
- 将结果和可选的检查点记录到 wandb 或 mlflow 中。

示例：

```shell
# finetune lora
accelerate launch -m axolotl.cli.train examples/openllama-3b/lora.yml
```

## [Llama-Factory](https://github.com/hiyouga/LLaMA-Factory/tree/main)

[<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003911543-740283500.png" alt="img" style="zoom:50%;" />](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003911543-740283500.png)

使用零代码命令行与 Web UI 轻松训练百余种大模型，并提供高效的训练和评估工具。

**主要特点：**

- **多种模型**：LLaMA、LLaVA、Mistral、Mixtral-MoE、Qwen、Yi、Gemma、Baichuan、ChatGLM、Phi 等等。
- **多种训练**：预训练、（多模态）指令监督微调、奖励模型训练、PPO/DPO/KTO/ORPO 训练等等。
- **多种精度**：16-bit全参微调、冻结微调、LoRA/QLoRA 微调。
- **先进算法**：GaLore、BAdam、DoRA、LongLoRA、LLaMA Pro、Mixture-of-Depths、LoRA+、LoftQ、PiSSA 和 Agent 微调。
- **实用技巧**：FlashAttention-2、Unsloth、RoPE scaling、NEFTune 和 rsLoRA。
- **实验监控**：LlamaBoard、TensorBoard、Wandb、MLflow 等等。
- **极速推理**：基于 vLLM 的 OpenAI 风格 API、浏览器界面和命令行接口。

示例：

```shell
llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml
llamafactory-cli chat examples/inference/llama3_lora_sft.yaml
llamafactory-cli export examples/merge_lora/llama3_lora_sft.yaml
```

## [Firfly](https://github.com/yangjianxin1/Firefly)

[<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003922003-1443343359.png" alt="img" style="zoom:50%;" />](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003922003-1443343359.png)

**Firefly** 支持对主流的大模型进行预训练、指令微调和 DPO。

**主要特点：**

- 支持预训练、SFT、DPO，支持全参数训练、LoRA/QLoRA 训练。
- 支持使用 [Unsloth](https://github.com/yangjianxin1/unsloth) 加速训练，降低显存需求。
- 支持绝大部分主流的开源大模型，如 Llama3、Gemma、MiniCPM、Llama、InternLM、Baichuan、ChatGLM、Yi、Deepseek、Qwen、Orion、Ziya、Xverse、Mistral、Mixtral-8x7B、Zephyr、Vicuna、Bloom，训练时与各个官方的 chat 模型的 template 对齐。
- 整理并开源指令微调数据集：firefly-train-1.1M 、moss-003-sft-data、ultrachat、 WizardLM_evol_instruct_V2_143k、school_math_0.25M。
- 在 Open LLM Leaderboard 上验证了 QLoRA 训练流程的有效性，开源[Firefly 系列指令微调模型权重](https://huggingface.co/YeungNLP)。

示例：

```shell
deepspeed --num_gpus={num_gpus} train.py --train_args_file train_args/sft/full/bloom-1b1-sft-full.json
torchrun --nproc_per_node={num_gpus} train.py --train_args_file train_args/pretrain/qlora/yi-6b-pretrain-qlora.json
```

## [Xtuner](https://github.com/InternLM/xtuner)

[<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003933062-1818906587.png" alt="img" style="zoom:50%;" />](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003933062-1818906587.png)

XTuner 由上海人工智能实验室发布，是一个高效、灵活、全能的轻量化大模型微调工具库。

**主要特点：**

- **高效**
  - 支持大语言模型 LLM、多模态图文模型 VLM 的预训练及轻量级微调。XTuner 支持在 8GB 显存下微调 7B 模型，同时也支持多节点跨设备微调更大尺度模型（70B+）。
  - 自动分发高性能算子（如 FlashAttention、Triton kernels 等）加速训练吞吐。
- **灵活**
  - 支持多种大语言模型，包括但不限于 [InternLM](https://huggingface.co/internlm)、[Mixtral-8x7B](https://huggingface.co/mistralai)、[Llama 2](https://huggingface.co/meta-llama)、[ChatGLM](https://huggingface.co/THUDM)、[Qwen](https://huggingface.co/Qwen)、[Baichuan](https://huggingface.co/baichuan-inc)，及多模态图文模型 LLaVA 的预训练与微调。
  - 兼容任意数据格式，开源数据或自定义数据皆可快速上手。
  - 支持增量预训练、[QLoRA](http://arxiv.org/abs/2305.14314)、[LoRA](http://arxiv.org/abs/2106.09685)、指令微调、Agent微调、全量参数微调等多种训练方式。
- **全能**
  - 预定义众多开源对话模版，支持与开源或训练所得模型进行对话。
  - 训练所得模型可无缝接入部署工具库 [LMDeploy](https://github.com/InternLM/lmdeploy)、大规模评测工具库 [OpenCompass](https://github.com/open-compass/opencompass) 及 [VLMEvalKit](https://github.com/open-compass/VLMEvalKit)。

示例：

```shell
xtuner train internlm2_5_chat_7b_qlora_oasst1_e3 --deepspeed deepspeed_zero2 # 单卡
# 多卡
(DIST) NPROC_PER_NODE=${GPU_NUM} xtuner train internlm2_5_chat_7b_qlora_oasst1_e3 --deepspeed deepspeed_zero2
(SLURM) srun ${SRUN_ARGS} xtuner train internlm2_5_chat_7b_qlora_oasst1_e3 --launcher slurm --deepspeed deepspeed_zero2
```

## [Swift](https://github.com/modelscope/ms-swift/tree/main)

[<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003943697-255792829.png" alt="img" style="zoom:50%;" />](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003943697-255792829.png)

ms-swift是魔塔提供的大模型与多模态大模型微调部署框架，支持450+大模型与150+多模态大模型的训练、推理、评测、量化与部署。

**主要特点：**

- 🍎 **模型类型**：支持450+纯文本大模型、**150+多模态大模型**，All-to-All全模态模型的**训练到部署全流程**。
- **数据集类型**：内置150+预训练、微调、人类对齐、多模态等各种类型的数据集，并支持自定义数据集。
- **多种训练：**
  - **轻量训练**：支持LoRA/QLoRA/DoRA/LoRA+/RS-LoRA、ReFT、LLaMAPro、Adapter、GaLore/Q-Galore、LISA、UnSloth、Liger-Kernel等轻量微调方式。支持对BNB、AWQ、GPTQ、AQLM、HQQ、EETQ量化模型进行训练。
  - **RLHF训练**：支持文本和多模态大模型的DPO、CPO、SimPO、ORPO、KTO、RM、PPO等RLHF训练。
  - **多模态训练**：支持对图像、视频和语音模态模型进行训练，支持VQA、Caption、OCR、Grounding任务的训练。
  - **界面训练**：以界面的方式提供训练、推理、评测、量化的能力，完成大模型的全链路。
- **插件化与拓展**：支持对loss、metric、trainer、loss-scale、callback、optimizer等组件进行自定义。
- **模型评测**：以EvalScope作为评测后端，支持100+评测数据集对纯文本和多模态模型进行评测。

示例：

```shell
CUDA_VISIBLE_DEVICES=0 swift sft --model Qwen/Qwen2.5-7B-Instruct \
    --train_type lora \
    --dataset 'AI-ModelScope/alpaca-gpt4-data-zh#500' \
    --lora_rank 8 --lora_alpha 32 \
    --target_modules all-linear \
    --warmup_ratio 0.05
```

## [Unsloth](https://github.com/unslothai/unsloth)

[<img src="https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003953386-1712899059.png" alt="img" style="zoom:50%;" />](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122003953386-1712899059.png)

Unsloth是一个开源的大模型训练加速项目，使用OpenAI的Triton对模型的计算过程进行重写，**大幅提升模型的训练速度，降低训练中的显存占用**。Unsloth能够保证重写后的模型计算的一致性，实现中不存在近似计算，**模型训练的精度损失为零**。

**主要特点：**

- 所有内核均使用OpenAI的Triton语言编写。采用手动反向传播引擎。
- 精度无损失——不采用近似方法——全部精确。
- 无需更改硬件。支持2018年及以后版本的NVIDIA GPU。最低CUDA Capability为7.0（V100、T4、Titan V、RTX 20/30/40、A100、H100、L40等）。GTX 1070、1080也可以使用，但速度较慢。
- 支持4bit和16bit GLorA/LoRA微调。
- 开源版本训练速度提高5倍，使用Unsloth Pro可获得高达30倍的训练加速！

示例：

```python
from unsloth import FastLanguageModel
# ... 导入其他包
max_seq_length = 2048 # Supports RoPE Scaling interally, so choose any!
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/llama-3-8b-bnb-4bit",
    max_seq_length = max_seq_length,
    dtype = None,
    load_in_4bit = True,
)
# 后续流程和使用 transformers.Trainer 类似
```

## [transformers.Trainer](https://huggingface.co/docs/transformers/zh/main_classes/trainer)

[![img](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122004004459-1119047213.png)](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122004004459-1119047213.png)

最后不得不提下大名鼎鼎的transformers库的Trainer，上述的很多工具其实也是在其基础上构建的。

Trainer本身是一个高度封装的类，但相比刚刚提到的工具，居然还有点偏底层了😅。

**主要特点：**

- **通用性：** Trainer是一个通用的训练接口，适用于各种NLP任务，如分类、回归、语言建模等。它提供了标准化的训练流程，使得用户无需从头开始编写训练代码。
- **灵活性**：用户可以通过自定义训练循环、损失函数、优化器、学习率调度器等方式来调整训练过程。
- **高级功能：** 混合精度训练、分布式训练、断点续训等。
- **自定义回调函数**：允许用户添加自定义回调函数，以便在训练过程的特定阶段执行自定义操作。

示例：

```python
from transformers import Trainer
# 加载模型、数据
trainer = Trainer(
    model,
    training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["validation"],
    data_collator=data_collator,
    tokenizer=tokenizer,
)
trainer.train()
```

## 总结

![img](https://img2024.cnblogs.com/blog/2338485/202501/2338485-20250122004026256-2093369919.png)