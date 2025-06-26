---
layout: post
title: "不同的解码策略"
date: 2025-06-16
tags: [LLM]
comments: true
author: zxy
---

在引人入胜的大型语言模型（LLM）世界中，模型架构、数据处理和优化备受关注。然而，在文本生成中发挥关键作用的解码策略，如beam search，却常被忽视。在本文中，我们将通过深入研究greedy search 、 beam search, 和 sampling techniques with top-k and nucleus sampling，探索 LLM 如何生成文本。

## Background

首先，让我们从一个例子开始。我们将文本“I have a dream”输入 GPT-2 模型，并要求它生成接下来的五个 token（单词或子词）。

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import torch

device = 'cuda' if torch.cuda.is_available() else 'cpu'
model = GPT2LMHeadModel.from_pretrained('gpt2').to(device)
tokenizer = GPT2Tokenizer.from_pretrained('gpt2')
model.eval()

text = "I have a dream"
input_ids = tokenizer.encode(text, return_tensors='pt').to(device)

outputs = model.generate(input_ids, max_length=len(input_ids.squeeze())+5)
generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(f"Generated text: {generated_text}")
```

```
Generated text: I have a dream of being a doctor.
```

人们普遍存在一个误解，认为像 GPT-2 这样的 LLMs 直接生成所有文本。事实并非如此，LLMs 计算 logits，即分配给其词汇表中每个可能 token 的分数。以下是该过程的说明性分解：

![image-20250616155728991](https://zxyandzxy.github.io/images/202506161557514.png)

分词器（在此例中为 Byte-Pair Encoding）将输入文本中的每个 token 转换为相应的 token ID。然后，GPT-2 使用这些 token ID 作为输入，并尝试预测下一个最可能的 token。最后，模型生成 logits，这些 logits 通过 softmax 函数转换为概率。

例如，模型将“of”作为“I have a dream”之后的下一个 token 的概率分配为 17%。这个输出本质上代表了序列中潜在下一个 token 的排名列表。更正式地说，我们将其概率表示为 $P(\text{of } | \text{ I have a dream}) = 17\% $。

自回归模型如 GPT 根据前一个词序列预测下一个词。考虑一个词序列 $w = (w_1, w_2, \ldots, w_t)$ ，这个序列的联合概率 P(w) 可以分解为：
$$
\begin{align} P(w) &= P(w_1, w_2, \ldots, w_t) \\ &= P(w_1) P(w_2 | w_1) P(w_3 | w_2, w_1) \ldots P(w_t | w_1, \ldots, w_{t-1}) \\ &= \prod_{i=1}^t P(w_i | w_1, \dots, w_{i-1}). \end{align}
$$
对于序列中的每个词 $w_i ， P(w_i | w_1, \ldots, w_{i-1})$ 表示在所有前序词$ (w_1, \ldots, w_{i-1})$ 的条件下 w_i 的条件概率。GPT-2 为其词汇表中的 50,257 个词计算这个条件概率。

这就引出了问题：我们如何利用这些概率来生成文本？这就是解码策略，如贪婪搜索和束搜索发挥作用的地方。

## Greedy Search

贪婪搜索是一种解码方法，它在每一步都选择最可能的标记作为序列中的下一个标记。简单来说，它只在每个阶段保留最可能的标记，丢弃所有其他潜在选项。以我们的例子为例：

- **Step 1**: Input: “I have a dream” → Most likely token: ” of”
- **Step 2**: Input: “I have a dream of” → Most likely token: ” being”
- **Step 3**: Input: “I have a dream of being” → Most likely token: ” a”
- **Step 4**: Input: “I have a dream of being a” → Most likely token: ” doctor”
- **Step 5**: Input: “I have a dream of being a doctor” → Most likely token: “.”

虽然这种方法听起来很直观，但需要注意的是贪婪搜索是短视的：它在每一步只考虑最可能的标记，而不考虑对整个序列的总体影响。这种特性使其快速高效，因为它不需要跟踪多个序列，但也意味着它可能会错过那些可能由稍低概率的下一个标记产生的更好序列。

接下来，我们使用 `graphviz` 和 `networkx` 来说明贪婪搜索的实现。我们选择得分最高的 ID，计算其日志概率（我们取对数以简化计算），并将其添加到树中。我们将重复此过程五次。

```python
import matplotlib.pyplot as plt
import networkx as nx
import numpy as np
import time

def get_log_prob(logits, token_id):
    # Compute the softmax of the logits
    probabilities = torch.nn.functional.softmax(logits, dim=-1)
    log_probabilities = torch.log(probabilities)
    
    # Get the log probability of the token
    token_log_probability = log_probabilities[token_id].item()
    return token_log_probability

def greedy_search(input_ids, node, length=5):
    if length == 0:
        return input_ids

    outputs = model(input_ids)
    predictions = outputs.logits

    # Get the predicted next sub-word (here we use top-k search)
    logits = predictions[0, -1, :]
    token_id = torch.argmax(logits).unsqueeze(0)

    # Compute the score of the predicted token
    token_score = get_log_prob(logits, token_id)

    # Add the predicted token to the list of input ids
    new_input_ids = torch.cat([input_ids, token_id.unsqueeze(0)], dim=-1)

    # Add node and edge to graph
    next_token = tokenizer.decode(token_id, skip_special_tokens=True)
    current_node = list(graph.successors(node))[0]
    graph.nodes[current_node]['tokenscore'] = np.exp(token_score) * 100
    graph.nodes[current_node]['token'] = next_token + f"_{length}"

    # Recursive call
    input_ids = greedy_search(new_input_ids, current_node, length-1)
    
    return input_ids

# Parameters
length = 5
beams = 1

# Create a balanced tree with height 'length'
graph = nx.balanced_tree(1, length, create_using=nx.DiGraph())

# Add 'tokenscore', 'cumscore', and 'token' attributes to each node
for node in graph.nodes:
    graph.nodes[node]['tokenscore'] = 100
    graph.nodes[node]['token'] = text

# Start generating text
output_ids = greedy_search(input_ids, 0, length=length)
output = tokenizer.decode(output_ids.squeeze().tolist(), skip_special_tokens=True)
print(f"Generated text: {output}")
```

```
Generated text: I have a dream of being a doctor.
```

我们的贪婪搜索生成的文本与 transformers 库生成的文本相同：“I have a dream of being a doctor.” 让我们可视化我们创建的树。

```python
import matplotlib.pyplot as plt
import networkx as nx
import matplotlib.colors as mcolors
from matplotlib.colors import LinearSegmentedColormap

def plot_graph(graph, length, beams, score):
    fig, ax = plt.subplots(figsize=(3+1.2*beams**length, max(5, 2+length)), dpi=300, facecolor='white')

    # Create positions for each node
    pos = nx.nx_agraph.graphviz_layout(graph, prog="dot")

    # Normalize the colors along the range of token scores
    if score == 'token':
        scores = [data['tokenscore'] for _, data in graph.nodes(data=True) if data['token'] is not None]
    elif score == 'sequence':
        scores = [data['sequencescore'] for _, data in graph.nodes(data=True) if data['token'] is not None]
    vmin = min(scores)
    vmax = max(scores)
    norm = mcolors.Normalize(vmin=vmin, vmax=vmax)
    cmap = LinearSegmentedColormap.from_list('rg', ["r", "y", "g"], N=256) 

    # Draw the nodes
    nx.draw_networkx_nodes(graph, pos, node_size=2000, node_shape='o', alpha=1, linewidths=4, 
                          node_color=scores, cmap=cmap)

    # Draw the edges
    nx.draw_networkx_edges(graph, pos)

    # Draw the labels
    if score == 'token':
        labels = {node: data['token'].split('_')[0] + f"\n{data['tokenscore']:.2f}%" for node, data in graph.nodes(data=True) if data['token'] is not None}
    elif score == 'sequence':
        labels = {node: data['token'].split('_')[0] + f"\n{data['sequencescore']:.2f}" for node, data in graph.nodes(data=True) if data['token'] is not None}
    nx.draw_networkx_labels(graph, pos, labels=labels, font_size=10)
    plt.box(False)

    # Add a colorbar
    sm = plt.cm.ScalarMappable(cmap=cmap, norm=norm)
    sm.set_array([])
    if score == 'token':
        fig.colorbar(sm, ax=ax, orientation='vertical', pad=0, label='Token probability (%)')
    elif score == 'sequence':
        fig.colorbar(sm, ax=ax, orientation='vertical', pad=0, label='Sequence score')
    plt.show()

# Plot graph
plot_graph(graph, length, 1.5, 'token')
```

![image-20250616162459450](https://zxyandzxy.github.io/images/202506161625546.png)

在这个图中，顶节点存储输入的 token（因此概率为 100%），而其他所有节点代表生成的 token。尽管这个序列中的每个 token 在预测时都是最可能的，但"being"和"doctor"分别被分配了相对较低的 9.68%和 2.86%的概率。这表明我们最初预测的 token "of"可能不是最合适的选择，因为它导致了不太可能的"being"。

## Beam Search

与仅考虑下一个最可能标记的贪婪搜索不同，Beam Search会考虑前 n 个最可能的标记，其中 n 代表集束的数量。这一过程会一直重复，直到达到预定义的最大长度或出现序列结束标记。此时，选择得分最高的序列（或“集束”）作为输出。

我们可以调整之前的函数，使其考虑概率最高的 n 个标记，而不仅仅是其中一个。在这里，我们将保持序列分数 $\log P(w)$ ，它是光束中每个标记对数概率的累积和。我们将此分数按序列长度进行归一化，以防止对较长的序列产生偏差（这个因子可以调整）。再次，我们将生成五个额外的token来完成句子“I have a dream”

```python
from tqdm.notebook import tqdm

def greedy_sampling(logits, beams):
    return torch.topk(logits, beams).indices
    
def beam_search(input_ids, node, bar, length, beams, sampling, temperature=0.1):
    if length == 0:
        return None

    outputs = model(input_ids)
    predictions = outputs.logits

    # Get the predicted next sub-word (here we use top-k search)
    logits = predictions[0, -1, :]

    if sampling == 'greedy':
        top_token_ids = greedy_sampling(logits, beams)
    elif sampling == 'top_k':
        top_token_ids = top_k_sampling(logits, temperature, 20, beams)
    elif sampling == 'nucleus':
        top_token_ids = nucleus_sampling(logits, temperature, 0.5, beams)

    for j, token_id in enumerate(top_token_ids):
        bar.update(1)

        # Compute the score of the predicted token
        token_score = get_log_prob(logits, token_id)
        cumulative_score = graph.nodes[node]['cumscore'] + token_score

        # Add the predicted token to the list of input ids
        new_input_ids = torch.cat([input_ids, token_id.unsqueeze(0).unsqueeze(0)], dim=-1)

        # Add node and edge to graph
        token = tokenizer.decode(token_id, skip_special_tokens=True)
        current_node = list(graph.successors(node))[j]
        graph.nodes[current_node]['tokenscore'] = np.exp(token_score) * 100
        graph.nodes[current_node]['cumscore'] = cumulative_score
        graph.nodes[current_node]['sequencescore'] = 1/(len(new_input_ids.squeeze())) * cumulative_score
        graph.nodes[current_node]['token'] = token + f"_{length}_{j}"

        # Recursive call
        beam_search(new_input_ids, current_node, bar, length-1, beams, sampling, 1)

# Parameters
length = 5
beams = 2

# Create a balanced tree with height 'length' and branching factor 'k'
graph = nx.balanced_tree(beams, length, create_using=nx.DiGraph())
bar = tqdm(total=len(graph.nodes))

# Add 'tokenscore', 'cumscore', and 'token' attributes to each node
for node in graph.nodes:
    graph.nodes[node]['tokenscore'] = 100
    graph.nodes[node]['cumscore'] = 0
    graph.nodes[node]['sequencescore'] = 0
    graph.nodes[node]['token'] = text

# Start generating text
beam_search(input_ids, 0, bar, length, beams, 'greedy', 1)
```

该函数计算了 63 个 token 和 beams^length = 2^5 = 32 种可能序列的分数。在我们的实现中，所有信息都存储在图中。我们的下一步是提取最佳序列。

首先，我们识别出具有最高序列分数的叶节点。接着，我们找到从根节点到该叶节点的最短路径。这条路径上的每个节点都包含最优序列中的一个标记。以下是我们如何实现它的方法：

```python
def get_best_sequence(G):
    # Create a list of leaf nodes
    leaf_nodes = [node for node in G.nodes() if G.out_degree(node)==0]

    # Get the leaf node with the highest cumscore
    max_score_node = None
    max_score = float('-inf')
    for node in leaf_nodes:
        if G.nodes[node]['sequencescore'] > max_score:
            max_score = G.nodes[node]['sequencescore']
            max_score_node = node

    # Retrieve the sequence of nodes from this leaf node to the root node in a list
    path = nx.shortest_path(G, source=0, target=max_score_node)

    # Return the string of token attributes of this sequence
    sequence = "".join([G.nodes[node]['token'].split('_')[0] for node in path])
    
    return sequence, max_score

sequence, max_score = get_best_sequence(graph)
print(f"Generated text: {sequence}")
```

```
Generated text: I have a dream. I have a dream
```

在这个可视化中，我们将展示每个节点的序列分数，该分数代表到目前为止序列的得分。如果函数 `get_best_sequence()` 是正确的，那么在序列“I have a dream。I have a dream”中的“dream”节点，应该在所有叶节点中得分最高。

![image-20250616164445025](https://zxyandzxy.github.io/images/202506161644922.png)

确实，“dream”这个 token 的序列得分最高，值为-0.69。有趣的是，我们可以看到左侧贪婪序列“I have a dream of being a doctor.”的得分为-1.16。

正如预期，贪婪搜索会导致次优结果。但坦白说，我们新的结果也不够吸引人。为了生成更多样化的序列，我们将实现两种采样算法：top-k 和 nucleus。

## Top-k sampling

Top-k 采样是一种利用语言模型生成的概率分布，从最可能的 k 个选项中随机选择一个标记的技术。

假设我们有 k = 3 和四个标记：A、B、C 和 D，相应的概率分别为： P(A) = 30% 、 P(B) = 15% 、 P(C) = 5% 和 P(D) = 1% 。在 top-k 采样中，标记 D 被忽略，算法将 60% 的概率输出 A，30% 的概率输出 B，10% 的概率输出 C。这种方法确保了我们优先考虑最可能的标记，同时在选择过程中引入了随机性。

引入随机性的另一种方式是温度的概念。温度 T 是一个介于 0 到 1 之间的参数，它影响 softmax 函数生成的概率，使最可能的词更有影响力。在实践中，它仅仅是将输入 logits 除以一个我们称为温度的值：$\text{softmax}(x_i) = \frac{e^{x_i / T}}{\sum_{j} e^{x_j / T}}$

这里有一个图表，展示了温度对给定输入 logits [1.5, -1.8, 0.9, -3.2]生成的概率的影响。我们绘制了三个不同的温度值，以观察差异。

![img](data:image/png;base64,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)

温度为 1.0 相当于没有温度的默认 softmax。另一方面，低温度设置（0.1）会显著改变概率分布。这通常用于文本生成，以控制生成输出的“创造性”水平。通过调整温度，我们可以影响模型产生更多样化或可预测响应的程度。

现在让我们实现 top k 采样算法。我们将通过提供“top_k”参数来在 `beam_search()` 函数中使用它。为了说明算法的工作原理，我们还将绘制 `top_k=20` 的概率分布图。

```python
def plot_prob_distribution(probabilities, next_tokens, sampling, potential_nb, total_nb=50):
    # Get top k tokens
    top_k_prob, top_k_indices = torch.topk(probabilities, total_nb)
    top_k_tokens = [tokenizer.decode([idx]) for idx in top_k_indices.tolist()]

    # Get next tokens and their probabilities
    next_tokens_list = [tokenizer.decode([idx]) for idx in next_tokens.tolist()]
    next_token_prob = probabilities[next_tokens].tolist()

    # Create figure
    plt.figure(figsize=(0.4*total_nb, 5), dpi=300, facecolor='white')
    plt.rc('axes', axisbelow=True)
    plt.grid(axis='y', linestyle='-', alpha=0.5)
    if potential_nb < total_nb:
        plt.axvline(x=potential_nb-0.5, ls=':', color='grey', label='Sampled tokens')
    plt.bar(top_k_tokens, top_k_prob.tolist(), color='blue')
    plt.bar(next_tokens_list, next_token_prob, color='red', label='Selected tokens')
    plt.xticks(rotation=45, ha='right', va='top')
    plt.gca().spines['top'].set_visible(False)
    plt.gca().spines['right'].set_visible(False)
    if sampling == 'top_k':
        plt.title('Probability distribution of predicted tokens with top-k sampling')
    elif sampling == 'nucleus':
        plt.title('Probability distribution of predicted tokens with nucleus sampling')
    plt.legend()
    plt.savefig(f'{sampling}_{time.time()}.png', dpi=300)
    plt.close()

def top_k_sampling(logits, temperature, top_k, beams, plot=True):
    assert top_k >= 1
    assert beams <= top_k

    indices_to_remove = logits < torch.topk(logits, top_k)[0][..., -1, None]
    new_logits = torch.clone(logits)
    new_logits[indices_to_remove] = float('-inf')

    # Convert logits to probabilities
    probabilities = torch.nn.functional.softmax(new_logits / temperature, dim=-1)

    # Sample n tokens from the resulting distribution
    next_tokens = torch.multinomial(probabilities, beams)

    # Plot distribution
    if plot:
        total_prob = torch.nn.functional.softmax(logits / temperature, dim=-1)
        plot_prob_distribution(total_prob, next_tokens, 'top_k', top_k)

    return next_tokens

# Start generating text
beam_search(input_ids, 0, bar, length, beams, 'top_k', 1)
```

![image-20250616170023889](https://zxyandzxy.github.io/images/202506161700411.png)

这些图表很好地展示了 top-k 采样是如何工作的，所有可能被选中的标记都位于水平条的左侧。虽然最可能的标记（红色）大部分时间被选中，但也允许不太可能的标记被选择。这提供了一个有趣的权衡，可以使序列倾向于一个不太可预测但听起来更自然的句子。现在让我们打印它生成的文本。

```python
sequence, max_score = get_best_sequence(graph)
print(f"Generated text: {sequence}")  # Generated text: I have a dream job and I want to
```

让我们看看这个决策树与上一个有何不同。

```python
# Plot graph
plot_graph(graph, length, beams, 'sequence')
```

![image-20250616170140366](https://zxyandzxy.github.io/images/202506161703139.png)

你可以看到这些节点与上一次迭代相比有显著差异，做出了更多样化的选择。尽管这个新结果的序列得分可能不是最高的（-1.01 而不是之前的-0.69），但重要的是要记住，更高的得分并不总是导致更真实或更有意义的序列。

现在我们已经介绍了 top-k 采样，接下来我们要介绍另一种最流行的采样技术：核采样。

## Nucleus sampling 

核采样，也称为 top-p 采样，与 top-k 采样采用了不同的方法。它不是选择概率最高的前 k 个 token，而是选择一个截止值 p ，使得所选 token 的概率之和超过 p 。这样就形成了一个“核”token 集合，从中随机选择下一个 token。

换句话说，模型按概率从高到低检查其 token，并将它们逐个添加到列表中，直到总概率超过阈值 p 。与 top-k 采样不同，核中包含的 token 数量每一步都可能不同。这种可变性通常能产生更多样化和富有创造力的输出，因此核采样在文本生成等任务中很受欢迎。

要实现核采样方法，我们可以使用 `beam_search()` 函数中的“核”参数。在这个例子中，我们将 p 的值设置为 0.5。为了简化，我们将包含一个最小数量的标记，等于束的数量。我们还将考虑累积概率低于 p 的标记，而不是高于。值得注意的是，虽然细节可能不同，但核采样的核心思想保持不变。

```python
def nucleus_sampling(logits, temperature, p, beams, plot=True):
    assert p > 0
    assert p <= 1

    # Sort the probabilities in descending order and compute cumulative probabilities
    sorted_logits, sorted_indices = torch.sort(logits, descending=True)
    probabilities = torch.nn.functional.softmax(sorted_logits / temperature, dim=-1)
    cumulative_probabilities = torch.cumsum(probabilities, dim=-1)

    # Create a mask for probabilities that are in the top-p
    mask = cumulative_probabilities < p

    # If there's not n index where cumulative_probabilities < p, we use the top n tokens instead
    if mask.sum() > beams:
        top_p_index_to_keep = torch.where(mask)[0][-1].detach().cpu().tolist()
    else:
        top_p_index_to_keep = beams

    # Only keep top-p indices
    indices_to_remove = sorted_indices[top_p_index_to_keep:]
    sorted_logits[indices_to_remove] = float('-inf')

    # Sample n tokens from the resulting distribution
    probabilities = torch.nn.functional.softmax(sorted_logits / temperature, dim=-1)
    next_tokens = torch.multinomial(probabilities, beams)

    # Plot distribution
    if plot:
        total_prob = torch.nn.functional.softmax(logits / temperature, dim=-1)
        plot_prob_distribution(total_prob, next_tokens, 'nucleus', top_p_index_to_keep)

    return next_tokens

# Start generating text
beam_search(input_ids, 0, bar, length, beams, 'nucleus', 1)
```

![image-20250616170928677](https://zxyandzxy.github.io/images/202506161709760.png)

在这个图中，你可以看到核中包含的 token 数量波动很大。生成的概率分布差异显著，导致选择的 token 并不总是最有可能的。这为生成独特且多样的序列打开了大门。现在，让我们看看它生成的文本。

```python
sequence, max_score = get_best_sequence(graph)
print(f"Generated text: {sequence}") # Generated text: I have a dream. I'm going to
```

要比较决策路径，让我们可视化新生成的树核采样。

![image-20250616171700207](https://zxyandzxy.github.io/images/202506161717245.png)

与 top-k 采样类似，这棵树与贪婪采样生成的树非常不同，显示出更多多样性。top-k 采样和核采样在生成文本时都提供了独特的优势，增强了多样性，并将创造力引入输出中。您在两种方法（甚至贪婪搜索）之间的选择将取决于您项目的具体需求和限制。

## Conclusion

在这篇文章中，我们深入探讨了 LLMs（特别是 GPT-2）所使用的各种解码方法。我们从简单的贪婪搜索开始，它立即选择最可能的下一个标记，尽管这通常不是最优的。接下来，我们介绍了束搜索技术，它在每一步考虑多个最可能的标记。虽然束搜索能提供更细致的结果，但它有时在生成多样化和富有创造性的序列方面会显得不足。

为了使过程更具多样性，我们随后介绍了 top-k 采样和核采样。top-k 采样通过随机选择 k 个最可能的标记来使文本生成多样化，而核采样则通过根据累积概率动态形成标记核来采取不同的路径。这些方法各自具有独特的优势和潜在缺点，而你的项目的具体需求将主要决定在这些方法之间的选择。













