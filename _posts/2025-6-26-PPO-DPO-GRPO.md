---
layout: post
title: "大模型优化利器：RLHF之PPO、DPO、GRPO"
date: 2025-06-26
tags: [LLM]
comments: true
author: zxy
---

# 大模型偏好对齐：从PPO到DPO

> 参考文章：[大模型优化利器：RLHF之PPO、DPO](https://www.zhihu.com/tardis/bd/art/717010380)

强化学习在大语言模型上的重要作用可以概括为以下几个方面：

- **强化学习比 SFT 更可以考虑整体影响**：SFT 针对单个 token 进行反馈，其目标是要求模型针对给定的输入给出的确切答案。而强化学习是针对整个输出文本进行反馈，并不针对特定的 token。这种反馈粒度的不同，使得强化学习更适合大语言模型，既可以兼顾表达多样性，还可以增强对微小变化的敏感性。自然语言十分灵活，可以用多种不同的方式表达相同的语义。而有监督学习很难支持上述学习方式。
- **强化学习更容易解决幻觉问题**：用户在大语言模型时主要有三类输入：（a）文本型：用户输入相关文本和问题，让模型基于所提供的文本生成答案；（b）求知型：用户仅提出问题，模型根据内在知识提供真实回答；（c）创造型（Creative）：用户为提供问题或说明，让模型进行创造性输出。SFT 非常容易使得求知型 query 产生幻觉。在模型并不包含或者知道答案的情况下，SFT 仍然会促使模型给出答案。而使用强化学习方法，则可以通过定制奖励函数，将正确答案赋予非常高的分数，放弃回答的答案赋予中低分数，不正确的答案赋予非常高的负分，使得模型学会依赖内部知识选择放弃回答，从而在一定程度上缓解模型幻觉问题。
- **强化学习可以更好的解决多轮对话奖励累积问题**：多轮对话能力是大语言模型重要的基础能力之一，多轮对话是否达成最终目标，需要考虑多次交互过程的整体情况，因此很难使用 SFT 方法构建。而使用强化学习方法，可以通过构建奖励函数，将当前输出考虑整个对话的背景和连贯性。

## RLHF

强化学习（Reinforcement Learning，RL）研究的问题是智能体（Agent）与环境（Environment） 交互的问题，其目标是使智能体在复杂且不确定的环境中最大化奖励（Reward）。强化学习基本框架如图 所示，主要由两部分组成：智能体和环境。在强化学习过程中，智能体与环境不断交互。

<img src="https://pic3.zhimg.com/v2-3b375dd479626f33ebc50dd7cba374fc_1440w.webp?consumer=ZHI_MENG" alt="img" style="zoom:50%;" />

智能体在环境中获取某个状态后，会根据该状态输出一个动作（Action），也称为决策（Decision）。 动作会在环境中执行，环境会根据智能体采取的动作，给出下一个状态以及当前动作所带来的奖励。智能体的目标就是尽可能多地从环境中获取奖励。

智能体在这个过程中不断学习，它的最终目标是：**找到一个策略，这个策略根据当前观测到的环境状态和奖励反馈，来选择最佳的动作。**

RLHF 主要分为**奖励模型训练**和**近端策略优化**两个步骤。奖励模型通过由人类反馈标注的偏好数据来学习人类的偏好，判断模型回复的有用性以及保证内容的无害性。奖励模型模拟了人类的偏好信息，能够不断地为模型的训练提供奖励信号。在获得奖励模型后，需要借助强化学习对语言模型继续进行微调。OpenAI 在大多数任务中使用的强化学习算法都是近端策略优化算法（Proximal Policy Optimization, PPO）。近端策略优化可以根据奖励模型获得的反馈优化模型，通过不断的迭代，让模型探索和发现更符合人类偏好的回复策略。PPO 的流程如图 所示。

![img](https://picx.zhimg.com/v2-35d7cb0cc53fc9f0c6756343019c3b0f_1440w.webp?consumer=ZHI_MENG)

PPO 涉及到四个模型：

- （1）**策略模型**（Policy Model），生成模型回复。
- （2）**奖励模型**（Reward Model），输出奖励分数来评估回复质量的好坏。
- （3）**评论模型**（Critic Model），来预测回复的好坏，可以在训练过程中实时调整模型，选择对未来累积收益最大的行为。
- （4）**参考模型**（Reference Model）提供了一个 SFT 模型的备份，帮助模型不会出现过于极端的变化。

PPO 的实施流程如下：

- (1) 环境采样：Policy Model 基于给定输入生成一系列的回复，Reward Model 则对这些回复进行打分获得奖励。
- (2) 优势估计：利用 Critic Model，预测生成回复的**未来累积奖励**，并借助广义优势估计（Generalized Advantage Estimation，GAE）算法来估计优势函数，能够有助于更准确地评估每次行动的好处。
- (3) 优化调整：使用优势函数来优化和调整 Policy Model，同时利用 Reference Model 确保更新的策略不会有太大的变化，从而维持模型的稳定性。

下图详细展示了 PPO 的整个流程：

![img](https://pic1.zhimg.com/v2-8499b498b656207243ee53b6e297eeb8_1440w.webp?consumer=ZHI_MENG)

### 奖励模型（Reward Model）

基于人类反馈训练的 Reward Model 可以很好的评估人类的偏好。从理论上来说，可以通过强化学习使用人类标注的反馈数据直接对模型进行微调。然而，受限于工作量和时间的限制，针对每次优化迭代，人类很难提供足够的反馈。更为有效的方法是构建 Reward Model，模拟人类的评估过程。Reward Model 在强化学习中起着至关重要的作用，它决定了智能体如何从与环境的交互中学习并优化策略，以实现预定的任务目标。

Reward Model 的输入是包含 chosen 和 rejected 的 pair 对，如下图所示。

<img src="https://picx.zhimg.com/v2-a186e30d69254b198699cd4f379487c7_1440w.webp?consumer=ZHI_MENG" alt="img" style="zoom:33%;" />

Reward Model 通常也采用基于 Transformer 架构的预训练语言模型。在 Reward Model 中，移除最后一个非嵌入层，并在最终的 Transformer 层上叠加了一个额外的线性层。无论输入的是何种文本，Reward Model 都能为文本序列中的最后一个 token 分配一个标量奖励值，样本质量越高，奖励值越大。

Reward Model 的损失定义如下：

 ![\mathcal{L}\left( \psi \right)=\text{log }\sigma\left( r\left( x,y_w \right) -r\left( x,y_l \right)\right)\tag3](https://www.zhihu.com/equation?tex=%5Cmathcal%7BL%7D%5Cleft%28+%5Cpsi+%5Cright%29%3D%5Ctext%7Blog+%7D%5Csigma%5Cleft%28+r%5Cleft%28+x%2Cy_w+%5Cright%29+-r%5Cleft%28+x%2Cy_l+%5Cright%29%5Cright%29%5Ctag3&consumer=ZHI_MENG) 

其中 ![\sigma](https://www.zhihu.com/equation?tex=%5Csigma&consumer=ZHI_MENG) 是 sigmoid 函数， ![r](https://www.zhihu.com/equation?tex=r&consumer=ZHI_MENG) 代表参数为 ![\psi](https://www.zhihu.com/equation?tex=%5Cpsi&consumer=ZHI_MENG) 的奖励模型的值， ![r\left( x,y \right)](https://www.zhihu.com/equation?tex=r%5Cleft%28+x%2Cy+%5Cright%29&consumer=ZHI_MENG) 表示针对输入提示 ![x](https://www.zhihu.com/equation?tex=x&consumer=ZHI_MENG) 和输出 ![y](https://www.zhihu.com/equation?tex=y&consumer=ZHI_MENG) 所预测出的单一标量奖励值。

### PPO

近端策略优化（[Proximal Policy Optimization, PPO](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1707.06347)）是对强化学习中策略梯度方法的改进，可以解决传统的策略梯度方法中存在的高方差、低数据效率、易发散等问题，从而提高了强化学习算法的可靠性和适用性。PPO 在各种基准任务中取得了非常好的性能，并且在机器人控制、自动驾驶、游戏玩家等领域中都有广泛的应用。OpenAI 在多个使用强化学习任务中都采用该方法，并将该方法成功应用于微调语言模型使之遵循人类指令和符合人类偏好。

> **策略梯度**

策略梯度方法有三个基本组成部分：演员（Actor）、环境和奖励函数，如下图所示，Actor 可以采取各种可能的动作与环境交互，在交互的过程中环境会依据当前环境状态和 Actor 的动作给出相应的奖励（Reward），并修改自身状态。Actor 的目的就在于调整策略（Policy），即根据环境信息决定采取什么动作以最大化奖励。

上述过程可以形式化的表示为：设环境的状态为 ![s_t](https://www.zhihu.com/equation?tex=s_t&consumer=ZHI_MENG) ，Actor 的策略函数 ![\pi_{\theta}](https://www.zhihu.com/equation?tex=%5Cpi_%7B%5Ctheta%7D&consumer=ZHI_MENG) 是从环境状态 ![s_t](https://www.zhihu.com/equation?tex=s_t&consumer=ZHI_MENG) 到动作 ![a_t](https://www.zhihu.com/equation?tex=a_t&consumer=ZHI_MENG) 的映射，其中 ![\theta](https://www.zhihu.com/equation?tex=%5Ctheta&consumer=ZHI_MENG) 是策略函数 ![\pi](https://www.zhihu.com/equation?tex=%5Cpi&consumer=ZHI_MENG) 的参数；奖励函数 ![r\left( s_t,a_t \right)](https://www.zhihu.com/equation?tex=r%5Cleft%28+s_t%2Ca_t+%5Cright%29&consumer=ZHI_MENG) 为从环境状态和 Actor 动作到奖励值的映射。一次完整的交互过程如图 所示。

<img src="https://pic1.zhimg.com/v2-d76f829878f00d6783e2fc4f5cf747c6_1440w.webp?consumer=ZHI_MENG" alt="img" style="zoom:33%;" />

策略梯度的基本形式如下：

![\begin{align} \nabla\bar{R}_{\theta} &=\mathbb{E}_{\tau \sim p_{\theta}\left( \tau \right)}\left[ R\left( \tau \right) \nabla \text{log }p_{\theta}\left( \tau \right) \right] \end{align} \tag{16}](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+%5Cnabla%5Cbar%7BR%7D_%7B%5Ctheta%7D+%26%3D%5Cmathbb%7BE%7D_%7B%5Ctau+%5Csim+p_%7B%5Ctheta%7D%5Cleft%28+%5Ctau+%5Cright%29%7D%5Cleft%5B+R%5Cleft%28+%5Ctau+%5Cright%29+%5Cnabla+%5Ctext%7Blog+%7Dp_%7B%5Ctheta%7D%5Cleft%28+%5Ctau+%5Cright%29+%5Cright%5D+%5Cend%7Balign%7D+%5Ctag%7B16%7D&consumer=ZHI_MENG)

![\theta \leftarrow \theta+\eta\nabla\bar{R}_{\theta}\tag{17}](https://www.zhihu.com/equation?tex=%5Ctheta+%5Cleftarrow+%5Ctheta%2B%5Ceta%5Cnabla%5Cbar%7BR%7D_%7B%5Ctheta%7D%5Ctag%7B17%7D&consumer=ZHI_MENG)

实际计算时，需要从环境中采样很多轨迹 ![\tau](https://www.zhihu.com/equation?tex=%5Ctau&consumer=ZHI_MENG) ，然后按照上述策略梯度公式对策略函数参数 ![\theta](https://www.zhihu.com/equation?tex=%5Ctheta&consumer=ZHI_MENG) 进行更新。但是由于 ![\tau](https://www.zhihu.com/equation?tex=%5Ctau&consumer=ZHI_MENG) 是从概率分布 ![p_{\theta}\left( \tau \right)](https://www.zhihu.com/equation?tex=p_%7B%5Ctheta%7D%5Cleft%28+%5Ctau+%5Cright%29&consumer=ZHI_MENG) 中采样得到，一旦策略函数参数 ![\theta](https://www.zhihu.com/equation?tex=%5Ctheta&consumer=ZHI_MENG) 更新，那么概率分布 ![p_{\theta}\left( \tau \right)](https://www.zhihu.com/equation?tex=p_%7B%5Ctheta%7D%5Cleft%28+%5Ctau+%5Cright%29&consumer=ZHI_MENG) 就会发生变化，因而之前采样过的轨迹便不能再次利用。所以策略梯度方法需要在不断地与环境交互中学习而不能利用历史数据。因而这种方法的训练效率低下。

策略梯度方法中，负责与环境交互的 Actor 与负责学习的 Actor 相同，这种训练方法被称为 **On-Policy** 训练方法。相反，**Off-Policy** 训练方法则将这两个 Actor 分离，固定一个 Actor 与环境交互而不更新它，而将交互得到的轨迹交由另外一个负责学习的 Actor 训练。Off-Policy 的优势是可以重复利用历史数据，从而提升训练效率。近端策略优化（Proximal Policy Optimization，PPO） 就是 Off-Policy。

> 此处省略一堆非常复杂的公式推导....................

![\begin{align} J^{\theta'}\left( \theta \right)_{\text{PPO}}&= J^{\theta'}\left( \theta \right)-\beta\text{ KL}\left( \pi_{\theta},\pi_{\theta'} \right)\\ J^{\theta'}\left( \theta \right) &=\mathbb{E}_{\left( s_t,a_t \right)\sim\pi_{\theta'}}\left[ \frac{p_{\theta}\left( s_t|a_t \right)}{p_{\theta'}\left( s_t|a_t \right)}A^{\theta'}\left( s_t,a_t \right)\right] \end{align} \tag{23}](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D+J%5E%7B%5Ctheta%27%7D%5Cleft%28+%5Ctheta+%5Cright%29_%7B%5Ctext%7BPPO%7D%7D%26%3D+J%5E%7B%5Ctheta%27%7D%5Cleft%28+%5Ctheta+%5Cright%29-%5Cbeta%5Ctext%7B+KL%7D%5Cleft%28+%5Cpi_%7B%5Ctheta%7D%2C%5Cpi_%7B%5Ctheta%27%7D+%5Cright%29%5C%5C+J%5E%7B%5Ctheta%27%7D%5Cleft%28+%5Ctheta+%5Cright%29+%26%3D%5Cmathbb%7BE%7D_%7B%5Cleft%28+s_t%2Ca_t+%5Cright%29%5Csim%5Cpi_%7B%5Ctheta%27%7D%7D%5Cleft%5B+%5Cfrac%7Bp_%7B%5Ctheta%7D%5Cleft%28+s_t%7Ca_t+%5Cright%29%7D%7Bp_%7B%5Ctheta%27%7D%5Cleft%28+s_t%7Ca_t+%5Cright%29%7DA%5E%7B%5Ctheta%27%7D%5Cleft%28+s_t%2Ca_t+%5Cright%29%5Cright%5D+%5Cend%7Balign%7D+%5Ctag%7B23%7D&consumer=ZHI_MENG)

公式（23)就是 PPO 最终的优化目标。

## DPO

前面我们详细介绍了 RLHF 的原理，整个过程略显复杂。首先需要训练好 Reward Model，然后在 PPO 阶段需要加载 4 个模型：Actor Model 、Reward Mode、Critic Model 和 Reference Model。对计算资源要求极高，而且训练时间长，对于一般人来说很难玩得起。

好在 2023 年 5 月，斯坦福大学提出了 PPO 的简化版：[DPO（Direct Preference Optimization）](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2305.18290)。只需要加载 2 个模型，而且不需要在线采样数据，极大地节省了训练开销。下图的右边是 DPO 的训练流程：

![img](https://pic2.zhimg.com/v2-44f397a445692fe8631990b251d10bdf_1440w.webp?consumer=ZHI_MENG)

下面就简单介绍下 DPO 是如何简化 PPO 的。首先回顾一下前面讲的 RLHF 是怎么训练的：

**第一步是训练 Reward Model**。训练数据是同一个 prompt 的 2 个回答，让人或 GPT-4o 标注哪个回答更好，Reward Model 会去优化如下目标：

![\mathcal{L}_{R}\left( r_{\phi},\mathcal{D} \right)=-\mathbb{E}_{(x,y_\text{win},y_\text{lose})\sim\mathcal{D}}[\log\sigma(r_\phi(x,y_\text{win})-r_\phi(x,y_\text{lose}))]\tag{24}](https://www.zhihu.com/equation?tex=%5Cmathcal%7BL%7D_%7BR%7D%5Cleft%28+r_%7B%5Cphi%7D%2C%5Cmathcal%7BD%7D+%5Cright%29%3D-%5Cmathbb%7BE%7D_%7B%28x%2Cy_%5Ctext%7Bwin%7D%2Cy_%5Ctext%7Blose%7D%29%5Csim%5Cmathcal%7BD%7D%7D%5B%5Clog%5Csigma%28r_%5Cphi%28x%2Cy_%5Ctext%7Bwin%7D%29-r_%5Cphi%28x%2Cy_%5Ctext%7Blose%7D%29%29%5D%5Ctag%7B24%7D&consumer=ZHI_MENG)

其中![r_\phi](https://www.zhihu.com/equation?tex=r_%5Cphi&consumer=ZHI_MENG)就是 Reward Model 用来给回答打分。![\mathcal{D}](https://www.zhihu.com/equation?tex=%5Cmathcal%7BD%7D&consumer=ZHI_MENG) 是训练数据集，![x](https://www.zhihu.com/equation?tex=x&consumer=ZHI_MENG) 是 prompt，![y_\text{win}](https://www.zhihu.com/equation?tex=y_%5Ctext%7Bwin%7D&consumer=ZHI_MENG) 和 ![y_\text{lose}](https://www.zhihu.com/equation?tex=y_%5Ctext%7Blose%7D&consumer=ZHI_MENG) 分别是好的回答和不好的回答。也就是说，要尽可能让好的回答的得分比不好的回答高，拉大他们之间的差别。

**第二步是用 RL 算法来提升模型的得分**。优化的目标是：

![\max_{\pi_\theta}\left\{\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[r_\phi(x,y)]-\beta\mathbb{D}_{\text{KL}}[\pi_\theta(y|x)||\pi_\text{ref}(y|x)]\right\}\tag{25}](https://www.zhihu.com/equation?tex=%5Cmax_%7B%5Cpi_%5Ctheta%7D%5Cleft%5C%7B%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5Br_%5Cphi%28x%2Cy%29%5D-%5Cbeta%5Cmathbb%7BD%7D_%7B%5Ctext%7BKL%7D%7D%5B%5Cpi_%5Ctheta%28y%7Cx%29%7C%7C%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29%5D%5Cright%5C%7D%5Ctag%7B25%7D&consumer=ZHI_MENG)

其中 ![\pi_\theta](https://www.zhihu.com/equation?tex=%5Cpi_%5Ctheta&consumer=ZHI_MENG) 是我们需要训练的 LLM，![\pi_\text{ref}](https://www.zhihu.com/equation?tex=%5Cpi_%5Ctext%7Bref%7D&consumer=ZHI_MENG) 是 Reference Model。这个优化目标的是希望 LLM 输出的回答的评分能尽可能高，同时 ![\pi_\theta](https://www.zhihu.com/equation?tex=%5Cpi_%5Ctheta&consumer=ZHI_MENG) 不要偏离 ![\pi_\text{ref}](https://www.zhihu.com/equation?tex=%5Cpi_%5Ctext%7Bref%7D&consumer=ZHI_MENG) 太多。

DPO 的作者们意识到，后面的这个式子是有显式解的。因为：

![\begin{aligned}\max_{\pi_\theta}&\left\{\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[r_\phi(x,y)] -\beta\mathbb{D}_{\text{KL}}[\pi_\theta(y|x)||\pi_\text{ref}(y|x)]\right\}\\&=\max_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[r_\phi(x,y) - \beta \log \frac{\pi_\theta(y|x)}{\pi_\text{ref}(y|x)}]\\&=\min_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[\log \frac{\pi_\theta(y|x)}{\pi_\text{ref}(y|x)} - \frac{1}{\beta} r_\phi(x,y)]\\&=\min_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[\log\frac{\pi_\theta(y|x)}{\pi_\text{ref}(y|x)e^{r_\phi(x,y)/\beta}}]\end{aligned}\tag{26}](https://www.zhihu.com/equation?tex=%5Cbegin%7Baligned%7D%5Cmax_%7B%5Cpi_%5Ctheta%7D%26%5Cleft%5C%7B%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5Br_%5Cphi%28x%2Cy%29%5D+-%5Cbeta%5Cmathbb%7BD%7D_%7B%5Ctext%7BKL%7D%7D%5B%5Cpi_%5Ctheta%28y%7Cx%29%7C%7C%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29%5D%5Cright%5C%7D%5C%5C%26%3D%5Cmax_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5Br_%5Cphi%28x%2Cy%29+-+%5Cbeta+%5Clog+%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29%7D%5D%5C%5C%26%3D%5Cmin_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5B%5Clog+%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29%7D+-+%5Cfrac%7B1%7D%7B%5Cbeta%7D+r_%5Cphi%28x%2Cy%29%5D%5C%5C%26%3D%5Cmin_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5B%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29e%5E%7Br_%5Cphi%28x%2Cy%29%2F%5Cbeta%7D%7D%5D%5Cend%7Baligned%7D%5Ctag%7B26%7D&consumer=ZHI_MENG)

如果我们归一化一下分母，即取 ![Z(x)=\sum_y\pi_\text{ref}(y|x)e^{r_\phi(x,y)/\beta}](https://www.zhihu.com/equation?tex=Z%28x%29%3D%5Csum_y%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29e%5E%7Br_%5Cphi%28x%2Cy%29%2F%5Cbeta%7D&consumer=ZHI_MENG)，也就可以构造出一个新的概率分布：

![\pi^*(y|x) = \pi_\text{ref}(y|x)e^{r_\phi(x,y)/\beta}/Z(x)\tag{27}](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A%28y%7Cx%29+%3D+%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29e%5E%7Br_%5Cphi%28x%2Cy%29%2F%5Cbeta%7D%2FZ%28x%29%5Ctag%7B27%7D&consumer=ZHI_MENG)

那么上式变成了：

![\begin{aligned}\min_{\pi_\theta}&\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[\log\frac{\pi_\theta(y|x)}{\pi_\text{ref}(y|x)e^{r_\phi(x,y)/\beta}}]\\ &=\min_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[\log\frac{\pi_\theta(y|x)}{\pi^*(y|x)}-\log Z(x)]\\ &=\min_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D},y\sim\pi_\theta(y|x)}[\log\frac{\pi_\theta(y|x)}{\pi^*(y|x)}]\\ &=\min_{\pi_\theta}\mathbb{E}_{x\sim \mathcal{D}}\mathbb{D}_\text{KL}(\pi_\theta(y|x)||\pi^*(y|x)) \end{aligned}\tag{28}](https://www.zhihu.com/equation?tex=%5Cbegin%7Baligned%7D%5Cmin_%7B%5Cpi_%5Ctheta%7D%26%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5B%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29e%5E%7Br_%5Cphi%28x%2Cy%29%2F%5Cbeta%7D%7D%5D%5C%5C+%26%3D%5Cmin_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5B%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi%5E%2A%28y%7Cx%29%7D-%5Clog+Z%28x%29%5D%5C%5C+%26%3D%5Cmin_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%2Cy%5Csim%5Cpi_%5Ctheta%28y%7Cx%29%7D%5B%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y%7Cx%29%7D%7B%5Cpi%5E%2A%28y%7Cx%29%7D%5D%5C%5C+%26%3D%5Cmin_%7B%5Cpi_%5Ctheta%7D%5Cmathbb%7BE%7D_%7Bx%5Csim+%5Cmathcal%7BD%7D%7D%5Cmathbb%7BD%7D_%5Ctext%7BKL%7D%28%5Cpi_%5Ctheta%28y%7Cx%29%7C%7C%5Cpi%5E%2A%28y%7Cx%29%29+%5Cend%7Baligned%7D%5Ctag%7B28%7D&consumer=ZHI_MENG)

由于 KL 散度在 2 个分布相等时取最小值，我们得到了这样的结论：RLHF 训练希望得到的最优的概率分布就是 ![\pi^*](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A&consumer=ZHI_MENG)。另一个角度来说，由 ![\pi^*](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A&consumer=ZHI_MENG) 的公式，我们相当于是得到了 ![r_\phi](https://www.zhihu.com/equation?tex=r_%5Cphi&consumer=ZHI_MENG) 和 ![\pi^*](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A&consumer=ZHI_MENG) 的关系，那么是否我们可以把训练 ![r_\phi](https://www.zhihu.com/equation?tex=r_%5Cphi&consumer=ZHI_MENG) 转化成直接去训练 ![\pi^*](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A&consumer=ZHI_MENG) 呢？简单转换一下 ![\pi^*](https://www.zhihu.com/equation?tex=%5Cpi%5E%2A&consumer=ZHI_MENG) 的定义式，可以得到：

![r_{\phi}(x,y)=\beta\log\frac{\pi^*(y|x)}{\pi_\text{ref}(y|x)}+\beta \log Z(x)\tag{29}](https://www.zhihu.com/equation?tex=r_%7B%5Cphi%7D%28x%2Cy%29%3D%5Cbeta%5Clog%5Cfrac%7B%5Cpi%5E%2A%28y%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y%7Cx%29%7D%2B%5Cbeta+%5Clog+Z%28x%29%5Ctag%7B29%7D&consumer=ZHI_MENG)

带入公式（25），也就有了：

![\max_{\pi^*}\left\{\mathbb{E}_{(x,y_\text{win},y_\text{lose})\sim\mathcal{D}}[\log\sigma(\beta\log\frac{\pi^*(y_\text{win}|x)}{\pi_\text{ref}(y_\text{win}|x)} - \beta\log\frac{\pi^*(y_\text{lose}|x)}{\pi_\text{ref}(y_\text{lose}|x)})]\right\}\tag{30}](https://www.zhihu.com/equation?tex=%5Cmax_%7B%5Cpi%5E%2A%7D%5Cleft%5C%7B%5Cmathbb%7BE%7D_%7B%28x%2Cy_%5Ctext%7Bwin%7D%2Cy_%5Ctext%7Blose%7D%29%5Csim%5Cmathcal%7BD%7D%7D%5B%5Clog%5Csigma%28%5Cbeta%5Clog%5Cfrac%7B%5Cpi%5E%2A%28y_%5Ctext%7Bwin%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Bwin%7D%7Cx%29%7D+-+%5Cbeta%5Clog%5Cfrac%7B%5Cpi%5E%2A%28y_%5Ctext%7Blose%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Blose%7D%7Cx%29%7D%29%5D%5Cright%5C%7D%5Ctag%7B30%7D&consumer=ZHI_MENG)

同样的，我们可以直接用这个优化目标去求 ![\pi_\theta](https://www.zhihu.com/equation?tex=%5Cpi_%5Ctheta&consumer=ZHI_MENG)：

![\max_{\pi_\theta}\left\{\mathbb{E}_{(x,y_\text{win},y_\text{lose})\sim\mathcal{D}}[\log\sigma(\beta\log\frac{\pi_\theta(y_\text{win}|x)}{\pi_\text{ref}(y_\text{win}|x)} - \beta\log\frac{\pi_\theta(y_\text{lose}|x)}{\pi_\text{ref}(y_\text{lose}|x)})]\right\}\tag{31}](https://www.zhihu.com/equation?tex=%5Cmax_%7B%5Cpi_%5Ctheta%7D%5Cleft%5C%7B%5Cmathbb%7BE%7D_%7B%28x%2Cy_%5Ctext%7Bwin%7D%2Cy_%5Ctext%7Blose%7D%29%5Csim%5Cmathcal%7BD%7D%7D%5B%5Clog%5Csigma%28%5Cbeta%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y_%5Ctext%7Bwin%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Bwin%7D%7Cx%29%7D+-+%5Cbeta%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y_%5Ctext%7Blose%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Blose%7D%7Cx%29%7D%29%5D%5Cright%5C%7D%5Ctag%7B31%7D&consumer=ZHI_MENG)

最终，我们可以得出 DPO 的 loss 如下所示：![\mathcal{L}_{\text{DPO}}\left( \pi_{\theta};\pi_{\text{ref}} \right)=-\mathbb{E}_{(x,y_\text{win},y_\text{lose})\sim\mathcal{D}}[\log\sigma(\beta\log\frac{\pi_\theta(y_\text{win}|x)}{\pi_\text{ref}(y_\text{win}|x)} - \beta\log\frac{\pi_\theta(y_\text{lose}|x)}{\pi_\text{ref}(y_\text{lose}|x)})]\tag{31}](https://www.zhihu.com/equation?tex=%5Cmathcal%7BL%7D_%7B%5Ctext%7BDPO%7D%7D%5Cleft%28+%5Cpi_%7B%5Ctheta%7D%3B%5Cpi_%7B%5Ctext%7Bref%7D%7D+%5Cright%29%3D-%5Cmathbb%7BE%7D_%7B%28x%2Cy_%5Ctext%7Bwin%7D%2Cy_%5Ctext%7Blose%7D%29%5Csim%5Cmathcal%7BD%7D%7D%5B%5Clog%5Csigma%28%5Cbeta%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y_%5Ctext%7Bwin%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Bwin%7D%7Cx%29%7D+-+%5Cbeta%5Clog%5Cfrac%7B%5Cpi_%5Ctheta%28y_%5Ctext%7Blose%7D%7Cx%29%7D%7B%5Cpi_%5Ctext%7Bref%7D%28y_%5Ctext%7Blose%7D%7Cx%29%7D%29%5D%5Ctag%7B31%7D&consumer=ZHI_MENG) 

这就是 DPO 的 loss。DPO 通过以上的公式转换把 RLHF 巧妙地转化为了 SFT，在训练的时候不再需要同时跑 4 个模型（Actor Model 、Reward Mode、Critic Model 和 Reference Model)，而是只用跑 Actor 和 Reference 2 个模型。

## DPO 变种

DPO 出来之后，由于其简单易用的特点，迅速成为大模型训练的标配，随后也出现了各种变种，比如 SimPO、Step-DPO、MCTS-DPO、SPO、Iterative-DPO。下面就以 Iterative-DPO 为例，介绍一下做了哪些改动。

### Iterative-DPO

[Iterative Reasoning Preference Optimization](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2404.19733) 是 2024 年 Meta 提出的 DPO 改进版。

思路很简单，下面介绍下具体流程：

- 首先训练一个 Reward Model。
- 然后将训练数据分成 m 等份（比如 3 等份），取第一份数据，用 LLM 对每个 prompt 采样出 k 个答案，使用 Reward Model 对 k 个回答进行打分，选出得分最高的和最低的，构建成 pair 对。使用第一份 pair 对数据训练一轮 DPO，更新 LLM。
- 再取第二份数据，用**更新后的 LLM** 对第二份数据的每个 prompt 采样出 k 个答案，使用 Reward Model 对 k 个回答进行打分，选出得分最高的和最低的，构建成 pair 对。使用**第二份 pair 对数据在更新后的 LLM** 上训练一轮 DPO。
- 重复以上过程，直到 m 份数据全部训练完成。

下图是 Iterative-DPO 的具体流程：

![img](https://pic3.zhimg.com/v2-d5bf8d5dbb07200a39df63b5762b27f0_1440w.webp?consumer=ZHI_MENG)

由于 Iterative DPO 在每轮训练完成后，都会基于最新模型重新采样数据，构建 pair 对，因此 Iterative DPO 是介于 Online-Policy 和 Offline-Policy 之间。

下图是 Iterative DPO 的两阶段流程：

<img src="https://picx.zhimg.com/v2-b8dec38b1b9b1bbfd7301deb54d52511_1440w.webp?consumer=ZHI_MENG" alt="img" style="zoom:50%;" />

# 从PPO到GRPO的演进

>参考文章：[大模型强化学习（GRPO、PPO、DPO） - 知乎](https://zhuanlan.zhihu.com/p/24240521221)

## DPO（直接偏好优化）

这个方法训练远比PPO算法训练难度低，PPO算法对资源消耗很大。

![img](https://pic2.zhimg.com/v2-a2cf5f95d8fa7c50a677882455df20ef_1440w.jpg)

PPO算法会多次采样，标注样本并不是只有好与不好两种样本，而是会存在多个样本，会有打分排序，还会引入奖励模型（**reward model**）。

DPO 可以直接依据策略来定义偏好损失。当存在一个关于模型响应的人类偏好数据集时，DPO 能够在训练过程中，使用简单的二元交叉熵目标来对策略进行优化，而无需明确地去学习奖励函数或者从策略中进行采样。

![img](https://pic4.zhimg.com/v2-28c7e1f000e6447a36d8c79daf2747ab_1440w.jpg)

DPO算法的优化目标更为简单，利用了从**奖励函数**到最优策略的解析映射，允许直接使用**人类偏好数据**进行简化的优化过程。

该目标增加了对偏好数据 yw 可能性，并减少非偏好 yl 可能性。

![img](https://pic2.zhimg.com/v2-cc81e93e0d9405972ed2714e350ed23b_1440w.jpg)

这公式其实有点像变种的奖励函数，通过 β 参数的权重来调节， πθ 参数的优化等效于在此变量更改下的奖励模型优化。

DPO的数据集主要是三部分组成**instruct prompt、chosen、rejected。**

![img](https://pic3.zhimg.com/v2-00c94509773ca9cf683a01584ec11240_1440w.jpg)

中文DPO数据就能很明显看出来，其实就是想让模型拟合我们期望的chosen数据，不要回答我设定的rejected数据。这种思想很像对比学习，拉开正负样本差距，让模型学习指定输出正确的回答。

DPO 是一种直接优化人类偏好的语言模型训练方法，通过简化传统 [RLHF](https://zhida.zhihu.com/search?content_id=253870169&content_type=Article&match_order=1&q=RLHF&zhida_source=entity) 流程（如奖励模型训练和强化学习阶段）来提升效率。以下从优缺点两个维度展开分析：

> **DPO 的核心优势**

**1、训练效率显著提升**

- - 省去奖励模型训练：DPO 绕过了 RLHF 中独立的奖励模型训练阶段，直接利用偏好数据调整模型策略，减少了计算成本和训练时间。
  - 单阶段优化：将问题转化为分类任务（最大化偏好数据似然），无需交替优化策略和奖励模型，简化了实现流程。

**2、算法稳定性更强**

- - 规避强化学习的高方差问题：传统 RLHF 依赖策略梯度方法（如 PPO），容易因奖励稀疏或噪声导致训练不稳定，而 DPO 通过概率匹配直接优化策略，降低方差。
  - 数学理论保障：DPO 基于偏好模型（如 Bradley - Terry）的严格数学推导，确保在理想数据下收敛到最优策略。

**3、实现复杂度低**

- - 无需复杂调参：RLHF 需平衡策略更新幅度、奖励模型置信度等超参数，DPO 仅需调整学习率和偏好权重，更易工程化。

**4、数据利用更高效**

- - 直接偏好建模：通过对比正 / 负样本对的偏好概率，更精准地捕捉人类意图，避免奖励模型因 “过度简化” 偏好关系导致的偏差。

> **DPO 的局限性**

**1、对数据质量高度敏感**

- - 依赖高质量偏好数据：若数据存在噪声（如标注不一致）或覆盖不全，模型易过拟合错误偏好。RLHF 可通过奖励模型泛化部分噪声，但 DPO 直接依赖原始数据。
  - 需大规模标注数据：DPO 的性能与偏好数据量强相关，标注成本可能成为瓶颈。

**2、灵活性受限**

- - 静态偏好假设：DPO 假设偏好关系固定（如 [Bradley - Terry 模型](https://zhida.zhihu.com/search?content_id=253870169&content_type=Article&match_order=1&q=Bradley+-+Terry+模型&zhida_source=entity)），难以动态适应复杂或分层的偏好（如多维度权衡）。
  - 难以引入先验知识：RLHF 允许手动调整奖励函数（如添加安全约束），而 DPO 完全由数据驱动，缺乏显式干预接口。

**3、可解释性较弱**

- 黑箱优化过程：DPO 直接调整策略参数，缺乏 RLHF 中奖励模型的中间信号，难分析模型决策依据，不利于调试。

**4、扩展性挑战**

- 多任务适配困难：在需同时优化多个目标（如安全性、创造性）时，RLHF 可通过组合多奖励函数灵活调整，而 DPO 需重新标注融合偏好的数据。
- 在线学习效率低：DPO 依赖离线批次数据，若需实时融入新反馈（如在线用户评分），需频繁重新训练，成本较高。

## GRPO

在对大型语言模型（LLM）进行微调的环节中，强化学习（RL）具有不可替代的重要作用。当前，广泛应用的近端策略优化（PPO）算法在应对大规模模型时，遭遇了沉重的计算和存储压力。鉴于 PPO 算法须要构建一个与策略模型规模大体相当的价值网络来对优势函数进行评估，所以在大模型场景下，这便引发了显著的内存占据以及计算成本。就拿在拥有数十亿乃至数千亿参数的语言模型上运用 PPO 来说，价值网络的训练和更新会耗费海量的计算资源，进而使得训练过程效率低下，难以实现扩展。

而且，PPO 算法在更新策略的过程中，极有可能致使策略分布产生剧烈变动，进而波及训练的稳定性。鉴于上述种种问题，[DeepSeek](https://zhida.zhihu.com/search?content_id=253870169&content_type=Article&match_order=1&q=DeepSeek&zhida_source=entity) 推出了一种创新的强化学习算法 —— 组相对策略优化（GRPO），其目标在于降低对价值网络的依赖程度，与此同时确保策略更新的稳定性与高效性。

<img src="https://pic1.zhimg.com/v2-88f04c9c365012f4c053aab965084216_1440w.jpg" alt="img" style="zoom:33%;" />

从上图可以看出来，GRPO减少了价值函数，有别于 PPO 需要像那样添加额外的价值函数近似，转而直接采用多个采样输出的平均奖励当作Baseline，这使得训练资源的使用量得到了显著削减。

去除value function , reward 直接对单个q生成的response进行打分，归一化后，作为替代的优势函数。

![img](https://pic1.zhimg.com/v2-c4ac0bc4931bf799d711e3a747933ca2_1440w.jpg)

上面公式，*ϵ* 和 *β* 是超参数，*A*^*i*,*t* 是根据每个组内的相对回报计算出来的优势值。GRPO 采用组相对的方式来计算优势，这和奖励模型的特点非常契合，因为奖励模型一般是用同一问题的不同输出进行比较的数据集来训练的。KL 散度不再加到奖励函数里，而是直接加在损失函数上。这样，优势函数 *A*^*i*,*t* 的计算复杂性就降低了。

然而**PPO算法**和**GRPO算法**的**KL散度**计算方式不同，使得每次计算的值都是正数。

![img](https://pic3.zhimg.com/v2-35ac63209f7129ad0b5b8568ebd2d344_1440w.jpg)

其中*A*^*i*,*t* 的计算方式，多次采样的奖励值，当前值减去一组奖励值平均除标准差，可以将奖励值转换为标准正态分布的形式。这有助于消除不同奖励值之间的量纲差异，使得奖励值在同一个尺度上进行比较。

这种相对优势的计算方式使得算法更加关注策略之间的相对表现，而不是绝对的奖励值。**GRPO 算法**通过直接使用相对奖励值和标准化奖励值来计算优势函数，减少了对价值网络的依赖

![img](https://pica.zhimg.com/v2-9d7282bcce8c2fbbd1e509bfde7327e2_1440w.jpg)



### 奖励值的计算方式

**PPO 算法**使用的是累积的折扣奖励来计算奖励值。简单来说，就是把未来的奖励按照一个折扣因子折算到当前时间步，然后把这些折算后的奖励累加起来。公式大概是这样的：

![img](https://pic3.zhimg.com/v2-a731fa5588c751967093cb814e3b6cfc_1440w.jpg)

这里 Rt 是时间步 t 的累积奖励，rt+k 是时间步 t+k 的即时奖励，γ 是折扣因子，取值范围是 0 到 1。PPO 还会用一个价值网络来估计每个状态的价值函数，这个价值函数用来计算优势函数，进而用于策略更新。

**GRPO 算法**则采用了一种相对奖励的方式。它不依赖传统的价值网络来计算奖励值，而是通过比较同一组内不同策略的相对表现来计算奖励值。具体来说，GRPO 计算的是每个策略相对于组内其他策略的相对优势，而不是绝对的累积奖励。这种方法减少了对价值网络的依赖，降低了计算复杂性。

### 策略更新方式

PPO 算法通过裁剪策略更新的比率来防止策略发生大幅度变化。它的目标函数中引入了一个剪切比率，约束新策略和旧策略之间的变化，避免策略剧烈更新导致性能下降。公式大概是这样的：

![img](https://pica.zhimg.com/v2-8a79152f3b22c2156d8e08723e4d4fc2_1440w.jpg)

这里 rt(θ) 是新策略和旧策略的比值，ε 是一个超参数，控制更新的步长。

**GRPO 算法**通过组相对的方式来计算优势，这和奖励模型的性质很契合，因为奖励模型通常是基于同一问题的不同输出进行比较的数据集来训练的。**GRPO** 直接把 KL 散度添加在损失函数上，而不是添加到奖励函数中，降低了优势函数计算的复杂性。

### 价值网络的使用

- **PPO 算法**需要维护一个与策略模型大小相当的价值网络来估计优势函数。这在大规模模型场景下会导致显著的内存占用和计算代价。
- **GRPO 算法**减少了对价值网络的依赖，直接使用多个采样输出的平均奖励作为基线，显著减少了训练资源的使用。

### 稳定性和效率

- **PPO 算法**通过引入策略更新约束来保证更新不会发生剧烈变化，提高了训练的稳定性和可靠性。但在处理大规模模型时可能会遇到计算和存储负担。
- **GRPO 算法**通过组相对的方式来计算优势，不仅减少了对价值网络的依赖，还保持了策略更新的稳定性和高效性。这使得 **GRPO 算法**在大规模模型场景下具有更好的扩展性和效率。

总的来说，**PPO 算法**和 **GRPO 算法**在计算奖励值和策略更新方面有显著差异。**PPO 算法**依赖累积的折扣奖励和价值网络来计算奖励值，而 **GRPO 算法**则采用相对奖励的方式，减少了对价值网络的依赖。**GRPO 算法**在大规模模型场景下具有更好的稳定性和效率。

## DPO vs. RLHF 关键对比 

| 维度       | DPO                               | RLHF                               |
| ---------- | --------------------------------- | ---------------------------------- |
| 训练流程   | 单阶段优化，无需奖励模型          | 两阶段（奖励模型训练 + 强化学习）  |
| 计算成本   | 低（省去奖励模型训练和 PPO 迭代） | 高（需额外训练奖励模型和策略优化） |
| 稳定性     | 高（避免策略梯度方差）            | 低（依赖 PPO 超参数调优）          |
| 数据依赖性 | 强（直接依赖标注偏好质量）        | 中等（奖励模型可部分泛化噪声）     |
| 灵活性     | 低（静态偏好建模）                | 高（支持动态调整奖励函数）         |
| 可解释性   | 弱（黑箱策略优化）                | 中（可通过奖励模型分析决策依据）   |

## 适用场景建议

- 优先选择 DPO ：

- - 标注资源充足，偏好关系简单且静态的任务（如文本风格迁移、简单对话生成）。
  - 追求快速迭代或算力受限的场景（如中小规模模型调优）。

- 优先选择 RLHF ：

- - 需动态调整目标（如平衡安全性与创造性）或多任务协同优化。
  - 数据噪声较多或需结合领域知识的复杂任务（如医疗、法律场景）。





