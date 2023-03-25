# 提示性应用

在本指南中，我们将介绍一些高级和有趣的方法，我们可以使用提示工程来执行有用和更高级的任务。

**注意，这部分内容正在大力开发中**。
主题：
- [生成数据](#生成数据)
- [程序辅助语言模型](#pal程序辅助语言模型)
- [python-notebooks](#python-notebooks)



---
## 生成数据
LLMs有很强的能力来生成文本。使用有效的提示策略可以引导模型产生更好、更一致、更符合事实的回答。LLMs在生成数据方面也特别有用，这对运行各种实验都很有用。例如，我们可以用它来为情感分类器生成快速样本，就像这样：

*提示:*
```
生成10个用于情感分析的范例。例子被分为正面或负面。产生2个负面例子和8个正面例子。使用这种格式的例子：
问：<句子>
答：<情绪>
```

*输出:*
```
问：我刚刚得到了一个最好的消息!
答：积极的

问：我们刚刚在工作中得到了加薪!
答：积极的

问：我为我今天的成就感到非常自豪。
答：积极的

问：我度过了有史以来最美好的一天!
答：积极的

问：我真的很期待周末的到来。
答：积极的

问：我刚刚收到了有史以来最好的礼物!
答：积极的

问：我现在很高兴。
答：积极的

问：我很庆幸有这样一个了不起的家庭。
答：积极的

问：外面的天气太阴沉了。
答：消极的

问：我刚刚得到一些可怕的消息。
答：消极的
```

这是非常有用的。我们在指南的另一部分中对一个不同的测试使用这个例子。

---

## PAL（程序辅助语言模型）
 
[Gao等人, (2022)](https://arxiv.org/abs/2211.10435)提出了一种方法，使用LLM来读取自然语言问题并生成程序作为中间推理步骤。被称为程序辅助语言模型（PAL），与思维链提示不同的是，它不是使用自由格式的文本来获得解决方案，而是将解决方案的步骤卸载到一个程序化的运行时间，如Python解释器。

![](./img/pal.png)

让我们看看一个使用LangChain和OpenAI GPT-3的例子。我们有兴趣开发一个简单的应用程序，它能够通过利用Python解释器来解释所问的问题并提供答案。

具体来说，我们有兴趣创建一个功能，允许使用LLM来回答那些需要理解日期的问题。我们将为LLM提供一个提示，其中包括一些从[这里](https://github.com/reasoning-machines/pal/blob/main/pal/prompt/date_understanding_prompt.py)采用的示例。 

这些是我们需要的进口：

```python
import openai
from datetime import datetime
from dateutil.relativedelta import relativedelta
import os
from langchain.llms import OpenAI
from dotenv import load_dotenv
```

让我们首先配置一些东西：

```python
load_dotenv()

# API configuration
openai.api_key = os.getenv("OPENAI_API_KEY")

# for LangChain
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
```

设置模型实例：

```python
llm = OpenAI(model_name='text-davinci-003', temperature=0)
```

设置提示+问题：

```python
question = "Today is 27 February 2023. I was born exactly 25 years ago. What is the date I was born in MM/DD/YYYY?"

DATE_UNDERSTANDING_PROMPT = """
# Q: 2015 is coming in 36 hours. What is the date one week from today in MM/DD/YYYY?
# If 2015 is coming in 36 hours, then today is 36 hours before.
today = datetime(2015, 1, 1) - relativedelta(hours=36)
# One week from today,
one_week_from_today = today + relativedelta(weeks=1)
# The answer formatted with %m/%d/%Y is
one_week_from_today.strftime('%m/%d/%Y')
# Q: The first day of 2019 is a Tuesday, and today is the first Monday of 2019. What is the date today in MM/DD/YYYY?
# If the first day of 2019 is a Tuesday, and today is the first Monday of 2019, then today is 6 days later.
today = datetime(2019, 1, 1) + relativedelta(days=6)
# The answer formatted with %m/%d/%Y is
today.strftime('%m/%d/%Y')
# Q: The concert was scheduled to be on 06/01/1943, but was delayed by one day to today. What is the date 10 days ago in MM/DD/YYYY?
# If the concert was scheduled to be on 06/01/1943, but was delayed by one day to today, then today is one day later.
today = datetime(1943, 6, 1) + relativedelta(days=1)
# 10 days ago,
ten_days_ago = today - relativedelta(days=10)
# The answer formatted with %m/%d/%Y is
ten_days_ago.strftime('%m/%d/%Y')
# Q: It is 4/19/1969 today. What is the date 24 hours later in MM/DD/YYYY?
# It is 4/19/1969 today.
today = datetime(1969, 4, 19)
# 24 hours later,
later = today + relativedelta(hours=24)
# The answer formatted with %m/%d/%Y is
today.strftime('%m/%d/%Y')
# Q: Jane thought today is 3/11/2002, but today is in fact Mar 12, which is 1 day later. What is the date 24 hours later in MM/DD/YYYY?
# If Jane thought today is 3/11/2002, but today is in fact Mar 12, then today is 3/1/2002.
today = datetime(2002, 3, 12)
# 24 hours later,
later = today + relativedelta(hours=24)
# The answer formatted with %m/%d/%Y is
later.strftime('%m/%d/%Y')
# Q: Jane was born on the last day of Feburary in 2001. Today is her 16-year-old birthday. What is the date yesterday in MM/DD/YYYY?
# If Jane was born on the last day of Feburary in 2001 and today is her 16-year-old birthday, then today is 16 years later.
today = datetime(2001, 2, 28) + relativedelta(years=16)
# Yesterday,
yesterday = today - relativedelta(days=1)
# The answer formatted with %m/%d/%Y is
yesterday.strftime('%m/%d/%Y')
# Q: {question}
""".strip() + '\n'
```

```python
llm_out = llm(DATE_UNDERSTANDING_PROMPT.format(question=question))
print(llm_out)
```

```python
exec(llm_out)
print(born)
```

这将输出以下内容： `02/27/1998`

---
## Python Notebooks

|描述|Notebooks|
|--|--|
|学习如何使用Python解释器与语言模型相结合来解决任务。|[程序辅助语言模型](../notebooks/pe-pal.ipynb)|

---

更多的例子即将出现!

[上一节（高级提示）](./prompts-advanced-usage.md)

[下一节（ChatGPT）](./prompts-chatgpt.md)