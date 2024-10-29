
![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029100515081-1662283358.png)


前一篇：《全面解释人工智能LLM模型的真实工作原理（三）》


序言： 本节作为整篇的收官之作，自然少不了与当今最先进的AI模型相呼应。这里我们将简单介绍全球首家推动人工智能生成人类语言的公司——OpenAI的GPT模型的基本原理。如果你也希望为人类的发展做出贡献，并投身于AI行业，这无疑是一个绝佳的起点。其他知识都是进入该行业的基础，而理解该模型是必须的。OpenAI的创始团队中包括科技巨头Elon Musk，以及2024年诺贝尔奖得主Geoffrey Hinton的学生伊利亚·苏茨克弗（Ilya Sutskever）。他们都是全球有钱又最具智慧和前瞻性的人物代表。OpenAI最初公开了ChatGPT\-2的语言模型（LLM）源代码，但在随后的ChatGPT\-3及之后的版本中停止了开源，逐渐背离了最初的开放承诺，导致公司内部核心成员的相继离开。本节介绍的模型由OpenAI参与者、现任斯坦福大学教授李飞飞的学生Andrej Karpathy基于ChatGPT3模型而来。


**（关注不迷路，及时收到最新的人工智能资料更新）**


**GPT架构**


接下来谈谈GPT架构。大多数GPT模型（尽管有不同的变化）都使用这种架构。如果你跟着文章读到这里，这部分应该相对容易理解。使用框图表示法，这就是GPT架构的高级示意图：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029100632239-405028040.png)


此时，除了“GPT Transformer块”，其他模块我们都已详细讨论过。这里的\+号只是表示两个向量相加（这意味着两个嵌入必须同样大小）。来看一下这个GPT Transformer块：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029100722204-1375155577.png)


就是这样。之所以称之为“Transformer”，是因为它源自并属于一种Transformer架构——我们将在下一节中详细了解。理解上没有影响，因为这里展示的所有模块我们都已讨论过。让我们回顾一下到目前为止构建这个GPT架构的过程：


• 我们了解到神经网络接收数字并输出其他数字，权重是可训练的参数


• 我们可以对这些输入/输出数字进行解释，赋予神经网络现实世界的意义


• 我们可以串联神经网络创建更大的网络，并可以将每一个称为“块”，用框来表示以简化图解。每个块的作用都是接收一组数字并输出另一组数字


• 我们学习了很多不同类型的块，每种块都有其不同的作用


• GPT只是这些块的一个特殊排列，如上图所示，解释方式在第一部分已讨论过


随着时间的推移，人们在此基础上做出了各种修改，使得现代LLM更加强大，但基本原理保持不变。


现在，这个GPT Transformer实际上在原始Transformer论文中被称为“解码器”。让我们看看这一点。


**Transformer架构**


这是驱动语言模型能力迅速提升的关键创新之一。Transformer不仅提高了预测准确性，还比先前的模型更高效（更容易训练），允许构建更大的模型。这是GPT架构的基础。


观察GPT架构，你会发现它非常适合生成序列中的下一个词。它基本遵循我们在第一部分讨论的逻辑：从几个词开始，然后逐个生成词。但是，如果你想进行翻译呢？比如，你有一句德语句子（例如“Wo wohnst du?” \= “Where do you live?”），你希望将其翻译成英语。我们该如何训练模型来完成这项任务？


第一步，我们需要找到一种输入德语单词的方法，这意味着我们要扩展嵌入，包含德语和英语。我猜一种简单的输入方式是将德语句子和生成的英文句子连接起来，并将其输入上下文。为了让模型更容易理解，我们可以添加一个分隔符。每一步看起来像这样：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029100847718-159048220.png)


这可以工作，但仍有改进空间：


• 如果上下文长度固定，有时会丢失原始句子


• 模型需要学习很多内容。包括两种语言，还需要知道是分隔符，它应该在此处开始翻译


• 每次生成一个词时，都需要处理整个德语句子，存在不同偏移。这意味着相同内容的内部表示不同，模型应该能够通过这些表示进行翻译


Transformer最初就是为此任务创建的，它由“编码器”和“解码器”组成——基本上是两个独立的模块。一个模块仅处理德语句子，生成中间表示（仍然是数值集合）——这被称为编码器。第二个模块生成单词（我们已经见过很多）。唯一的区别是，除了将已生成的单词输入解码器外，还将编码器输出的德语句子作为额外输入。也就是说，在生成语言时，它的上下文是已生成的所有单词加上德语句子。这个模块被称为解码器。


这些编码器和解码器由一些块组成，尤其是夹在其他层之间的注意力块。我们来看“Attention is all you need”论文中的Transformer架构示意图并尝试理解它：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029100949657-530682653.png)


左侧的竖直块集合称为“编码器”，右侧的称为“解码器”。让我们逐个理解每个部分：


前馈网络：前馈网络是没有循环的网络。第一部分中讨论的原始网络就是一个前馈网络。事实上，这个块采用了非常相似的结构。它包含两个线性层，每个层之后都有一个ReLU（见第一部分关于ReLU的介绍）和一个Dropout层。请记住，这个前馈网络适用于每个位置独立。也就是说，位置0有一个前馈网络，位置1有一个，依此类推。但是位置x的神经元不会与位置y的前馈网络相连。这样做的重要性在于防止网络在训练时“偷看”前方信息。


交叉注意力：你会注意到解码器有一个多头注意力，其箭头来自编码器。这里发生了什么？记得自注意力和多头注意力中的value、key、query吗？它们都来自同一个序列。事实上，query只是序列的最后一个词。那么，如果我们保留query，但将value和key来自一个完全不同的序列会怎样？这就是这里发生的情况。value和key来自编码器的输出。数学上没有任何改变，只是key和value的输入来源发生了变化。


Nx：Nx表示这个块重复N次。基本上，你在将一个块层层堆叠，前一个块的输出作为下一个块的输入。这样可以使神经网络更深。从图上看，编码器输出如何传递给解码器可能让人困惑。假设N\=5。我们是否将每层编码器输出传递给对应的解码器层？不是的。实际上你只需运行一次编码器，然后将同一表示提供给5个解码器层。


加与归一化块：这与下方相同（作者似乎只是为了节省空间）。


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029101101062-2078543530.png)


其他内容我们已经讨论过。现在你已经完整理解了Transformer架构，从简单的加法和乘法操作一步步构建到现在的完整自包含解释！你知道如何从头构建Transformer的每一行、每一加法、每一块和每个单词的意义。如果你感兴趣，可以参看这个开源库（开源GPT: [https://github.com/karpathy/nanoGPT），它从头实现了上述的GPT架构。](https://github.com)


**附录**


**矩阵乘法**


在嵌入部分中，我们引入了向量和矩阵的概念。矩阵有两个维度（行数和列数）。向量也可以看作一个只有一个维度的矩阵。两个矩阵的乘积定义为：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029101154168-1682471702.png)


点表示相乘。现在我们再看一下第一张图中蓝色和有机神经元的计算。如果我们将权重写成矩阵，输入作为向量，可以将整个运算表示如下：
![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241029101239766-2108473882.png)


如果权重矩阵称为“W”，输入称为“x”，则Wx为结果（在此情况下是中间层）。我们也可以将两者转置写作xW——这是个人偏好的问题。


**标准差**


在层归一化部分，我们使用了标准差的概念。标准差是一个统计量，用于描述数值的分布范围（在一组数字中），例如，如果所有值都相同，则标准差为零。如果每个值都与这些值的均值相距很远，则标准差会很高。计算一组数字a1, a2, a3…（假设有N个数字）的标准差的公式如下：将每个数字减去均值，然后将每个N个数字的结果平方。将所有这些数字相加，然后除以N，最后对结果开平方根。


**位置编码**


我们在上文中提到过位置嵌入。位置编码与嵌入向量长度相同，不同之处在于它不是嵌入，且无需训练。我们为每个位置分配一个独特的向量。例如，位置1是一个向量，位置2是另一个，以此类推。


**(完结)**


**欢迎大家在评论区沟通讨论，作者同样可以为您解释模型当中的全部原理和实现过程与细节。**


 本博客参考[westworld加速](https://tianchuang88.com)。转载请注明出处！
