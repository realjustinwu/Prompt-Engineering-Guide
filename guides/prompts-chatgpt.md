# ChatGPT提示工程

在本节中，我们将介绍ChatGPT的最新提示工程技术，包括提示、应用、限制、论文和其他阅读材料。

**请注意，本节正在大力发展中**。

主题：
- [ChatGPT介绍](#chatgpt简介)
- [回顾对话任务](#回顾对话任务)
- [与ChatGPT的对话](#与chatgpt的对话)
- [Python notebooks](#python-notebooks)

---
## ChatGPT简介

ChatGPT是一个新的模型[由OpenAI训练](https://openai.com/blog/chatgpt)，可以进行对话式互动。这个模型被训练成能够遵循提示中的指令，在对话的背景下提供适当的回应。ChatGPT可以帮助回答问题、建议食谱、用某种风格写歌词、生成代码等等。

ChatGPT是使用人类反馈强化学习（RLHF）来训练的。虽然这个模型比以前的GPT迭代能力强很多（而且还经过训练以减少有害和不真实的输出），但它仍然有局限性。让我们通过具体的例子来介绍一些能力和限制。

你可以使用ChatGPT的研究预览[这里](https://chat.openai.com)，但对于下面的例子，我们将使用OpenAI Playground上的`聊天'模式。

---
## 回顾对话任务

在[以前的指南](prompts-basic-usage.md#对话)中，我们介绍了一些关于对话能力和角色提示的情况。我们介绍了如何指示LLM以特定的风格进行对话，有特定的意图、行为和身份。

让我们回顾一下我们之前的基本例子，我们创建了一个对话系统，能够对问题产生更多的技术和科学回答。

*提示:*
```
下面是与一个AI的对话。助理的语气是技术和科学的。

人类：你好，你是谁？
AI：你好! 我是一名AI。今天我能为您提供什么帮助？
人类： 你能告诉我关于黑洞的产生吗？
AI：
```

从上面的例子中，你可以看到两个重要的组成部分：
- 聊天机器人的**意图**或解释是什么
- **身份**，指示聊天机器人将使用何种风格或语气来回应

上面的简单例子在使用 "text-davinci-003 "的文本补全API中运行良好。最近，OpenAI[宣布了ChatGPT APIs](https://openai.com/blog/introducing-chatgpt-and-whisper-apis)，这是一个更强大和更便宜的模型，称为`gpt-3.5-turbo`是专门为这种类型的功能（聊天完成）而建立的。OpenAI推荐这是他们的最佳模型，即使是非聊天用例。 使用ChatGPT APIs的其他好处是大大降低了成本（90%）和效率。

像Snap Inc.和Instacart这样的大公司已经在他们的产品上整合了由ChatGPT驱动的对话功能，范围从个性化推荐到开放式的购物目标。

---
## 与ChatGPT的对话

### 多轮对话

为了开始演示ChatGPT的能力，我们将使用上面的聊天机器人助手的例子，并讨论其结果。与`text-davinci-003`相比，支持ChatGPT的`gpt-3.5-turbo`模型使用聊天格式作为输入。该模型期望有一系列的信息作为输入，并使用这些信息来生成一个响应。

*输入:*
```
系统：你是一名AI研究助理。你使用的语气是技术和科学的。
用户：你好，你是谁？
助理：你好! 我是一名AI研究助理。我今天能帮你什么忙吗？
用户：你能告诉我关于黑洞的产生吗？
```

*输出:*
```
助手：当然了! 当一颗巨大的恒星耗尽燃料并在其自身的引力下坍缩时，黑洞就会产生。这种坍缩导致恒星变得异常密集，其引力场如此强大，甚至连光都无法逃脱。这个不归点被称为事件视界。一旦一颗恒星越过事件视界，它就被认为是一个黑洞。
```

请注意，在上面的例子中，我简化了输入和输出，但ChatGPT的聊天完成API要求消息必须是特定的格式。我在下面添加了一个快照，说明这个例子在OpenAI Playground中使用 "聊天模式 "会是什么样子：

![](./img/chatgpt-1.png)

对于我们的例子，更正式的API调用看起来就像下面的例子：

```python
import openai

openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are an AI research assistant. You use a tone that is technical and scientific."},
        {"role": "user", "content": "Hello, who are you?"},
        {"role": "assistant", "content": "Greeting! I am an AI research assistant. How can I help you today?"},
        {"role": "user", "content": "Can you tell me about the creation of black holes?"}
    ]
)
```
未来开发者与ChatGPT的互动方式有望通过[Chat Markup Language](https://github.com/openai/openai-python/blob/main/chatml.md)（简称ChatML）完成。

### 单次转弯任务

聊天格式可以实现多轮对话，但它也支持单轮任务，与我们在`text-davinci-003'中使用的类似。这意味着我们可以使用ChatGPT来执行与我们为原始GPT模型所演示的类似的任务。例如，让我们尝试用ChatGPT来执行以下的问答任务：

*输入:*
```
用户：根据下面的上下文回答问题。回答要简短、扼要。如果不确定答案，请回答 "不确定答案"。

语境： 泰普利珠单抗的历史可以追溯到新泽西州一家名为Ortho Pharmaceutical的药物公司。在那里，科学家们产生了一种早期版本的抗体，被称为OKT3。该分子最初来自于小鼠，能够与T细胞表面结合并限制其细胞杀伤潜力。1986年，它被批准用于帮助防止肾脏移植后的器官排斥，使其成为第一个允许人类使用的治疗性抗体。

问题： OKT3最初的来源是什么？

回答：
```

*输出:*
```
助手：小鼠。
```

请记住，我添加`USER'和`ASSISTANT'的标签是为了更好地演示如何使用ChatGPT执行任务。下面是使用Playground的例子：

![](./img/chatgpt-classic.png)

更正式地说，这是API调用（我只包括请求的消息部分）：

```python
CONTENT = """根据下面的上下文回答问题。回答要简短、扼要。如果不确定答案，请回答 "不确定答案"。

语境： 泰普利珠单抗的历史可以追溯到新泽西州一家名为Ortho Pharmaceutical的药物公司。在那里，科学家们产生了一种早期版本的抗体，被称为OKT3。该分子最初来自于小鼠，能够与T细胞表面结合并限制其细胞杀伤潜力。1986年，它被批准用于帮助防止肾脏移植后的器官排斥，使其成为第一个允许人类使用的治疗性抗体。

问题： OKT3最初的来源是什么？

回答：
"""

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": CONTENT},
    ],
    temperature=0,
)
```

### 指导性的聊天模型

根据OpenAI的官方文档，`gpt-3.5-turbo`模型的快照也将被提供。例如，我们可以访问3月1日的快照`gpt-3.5-turbo-0301`。这使得开发人员可以选择特定的模型版本。这也意味着指导模型的最佳实践可能会在不同的版本中发生变化。

目前对`gpt-3.5-turbo-0301`的建议是在`用户`信息中添加指示，而不是可用的`系统`信息。

---
## Python Notebooks

|描述|Notebooks|
|--|--|
|了解如何使用 "openai "库调用ChatGPT的API。
|了解如何使用 "LangChain "库来使用ChatGPT的功能。|[使用LangChain的ChatGPT API](../notebooks/pe-chatgpt-langchain.ipynb)|

---
## 参考文献

- [通过学生的眼睛看ChatGPT：对TikTok数据的分析](https://arxiv.org/abs/2303.05349) (2023年3月)
- [用对话语言模型和提示工程从研究论文中提取准确的材料数据--ChatGPT的例子](https://arxiv.org/abs/2303.05352) (2023年3月)
- [ChatGPT呼之欲出： 一个大型的语言模型可能是我们对智能交通的全部需求吗？](https://arxiv.org/abs/2303.05382) (2023年3月)
- [做一个计算型律师](https://arxiv.org/abs/2303.05383) (2023年3月)
- [LLMs的合成数据生成是否有助于临床文本挖掘？](https://arxiv.org/abs/2303.04360) (2023年3月)
- [MenuCraft：用大型语言模型设计交互式菜单系统](https://arxiv.org/abs/2303.04496) (2023年3月)
- [A Comprehensive Survey of AI-Generated Content (AIGC)： 从GAN到ChatGPT的生成性人工智能的历史](https://arxiv.org/abs/2303.04226) (2023年3月)
- [探索ChatGPT在事件提取方面的可行性](https://arxiv.org/abs/2303.03836)
- [ChatGPT：人工注释的终结之始？自动流派识别的用例](https://arxiv.org/abs/2303.03953) (2023年3月)
- [ChatGPT是一个好的NLG评估器吗？ 初步研究](https://arxiv.org/abs/2303.04048) (2023年3月)
- [情感计算会从基础模型和通用人工智能中出现吗？对ChatGPT的首次评估](https://arxiv.org/abs/2303.03186) (2023年3月)
- [UZH_CLyp在SemEval-2023上的任务9：在Tweet亲密关系预测中用于跨语言学习的先头微调和ChatGPT数据生成](https://arxiv.org/abs/2303.01194) (2023年3月)
- [如何格式化ChatGPT模型的输入](https://github.com/openai/openai-cookbook/blob/main/examples/How_to_format_inputs_to_ChatGPT_models.ipynb) (2023年3月)
- [ChatGPT能评估人的个性吗？ 一个一般的评估框架](https://arxiv.org/abs/2303.01248) (2023年3月)
- [通过ChatGPT进行跨语言总结](https://arxiv.org/abs/2302.14229) (2023年2月)
- [聊天室： 利用ChatGPT进行文本数据扩展](https://arxiv.org/abs/2302.13007) (Feb 2023)
- [ChatGPT博士，告诉我我想听的东西： 提示知识如何影响健康答案的正确性](https://arxiv.org/abs/2302.13793) (2023年2月)
- [ChatGPT对数学单词问题（MWP）的独立评估](https://arxiv.org/abs/2302.13814) (Feb 2023)
- [ChatGPT：2.5个月后的元分析](https://arxiv.org/abs/2302.13795) (Feb 2023)
- [让我们聊一聊! 与ChatGPT的对话： 技术、应用和局限性](https://arxiv.org/abs/2302.13817) (2023年2月)
- [检查你的事实，并再次尝试： 利用外部知识和自动反馈改进大型语言模型](https://arxiv.org/abs/2302.12813) (2023年2月)
- [论ChatGPT的稳健性： 一个对抗性和分布外的视角](https://arxiv.org/abs/2302.12095) (2023年2月)
- [诸如ChatGPT这样的生成性人工智能模型如何在SPC实践、教育和研究中被（误）用？一个探索性研究](https://arxiv.org/abs/2302.10916) (2023年2月)
- [聊天工具也能听懂吗？关于ChatGPT和微调BERT的比较研究](https://arxiv.org/abs/2302.10198) (Feb 2023)
- [用ChatGPT加强提示工程的提示模式目录](https://arxiv.org/abs/2302.11382) (2023年2月)
- [通过ChatGPT的聊天进行零距离信息提取](https://arxiv.org/abs/2302.10205) (2023年2月)
- [ChatGPT：万能的，无能的](https://arxiv.org/abs/2302.10724) (2023年2月)
- [ChatGPT和DALL-E 2在决策和空间推理方面的试点评估](https://arxiv.org/abs/2302.09068) (2023年2月)
- [网民、学者和信息专家对人工智能的看法，特别是对ChatGPT的看法](https://arxiv.org/abs/2302.07136) (2023年2月)
- [ChatGPT中的语言歧义分析](https://arxiv.org/abs/2302.06426) (2023年2月)
- [ChatGPT与知识图谱的传统答题方法： 知识图谱聊天机器人的现状和未来方向](https://arxiv.org/abs/2302.06466) (2023年2月)
- [ChatGPT和生成性人工智能对科学意味着什么](https://www.nature.com/articles/d41586-023-00340-6) (2023年2月)
- [应用BERT和ChatGPT对科学文献中的莱姆病进行情感分析](https://arxiv.org/abs/2302.06474) (Feb 2023)
- [探索ChatGPT的AI伦理：诊断分析](https://arxiv.org/abs/2301.12867) (2023年1月)
- [探索ChatGPT的AI伦理：诊断性分析](https://arxiv.org/abs/2301.12867) (2023年1月)
- [ChatGPT for Good? 论大型语言模型对教育的机遇和挑战](https://www.edu.sot.tum.de/fileadmin/w00bed/hctl/_my_direct_uploads/ChatGPT_for_Good_.pdf) (2023年1月)
- [对话式人工智能的政治意识形态：关于ChatGPT的亲环境、左翼自由主义倾向的证据趋于一致](https://arxiv.org/abs/2301.01768) (2023年1月)
- [提高可靠性的技术--OpenAI Cookbook](https://github.com/openai/openai-cookbook/blob/main/techniques_to_improve_reliability.md)
- [令人敬畏的ChatGPT提示](https://github.com/f/awesome-chatgpt-prompts)
- [介绍ChatGPT](https://openai.com/blog/chatgpt) (2022年11月)

---
[上一节（应用）](./prompts-applications.md)

[下一节（对抗性提示）](./prompts-adversarial.md)
