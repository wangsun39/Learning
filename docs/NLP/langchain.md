

2025/4/22

[LangChain for LLM Application Development](https://learn.deeplearning.ai/courses/langchain/lesson/ls57z/memory)

[latex](https://www.latexlive.com/)

## LangChain for LLM Application Development
LLM 实际上是没有记忆的(stateless)，每次交互是独立的，
Chatbots appear to have memory by providing the full conversation as context.
一些管理记忆的方法

- buffer window 指定记忆消息条数
- token buffer 指定记忆token数
- Summary buffer 对话转成摘要

### QA系统
将LLM和未训练过的数据相结合

#### Embeddings 和 向量数据库

**Embeddings**: 一段文字转成一个向量, 相似的文本具有相似的向量

**向量数据库**: 存储向量的数组，并不在LLM里面，因为很大，也不能一次都传给LLM

大文本拆分成小块，构成向量数据库

把 query 转成 向量，在向量数据库中搜索最接近的前k个句子，把这k个句子合并起来，加上问题，发给LLM，让它给出一个合适的答案，相当于把问题和可能信息给LLM，让它从中提取信息。

另外有一些组织k个搜索结果的句子的方式，当句子非常大时可能有用

- map_reduce
- refine
- map_rerank


### 评估
可以手工编写一个问题和答案

也可以通过langchain来根据自动构造一些问答
```python
from langchain.evaluation.qa import QAGenerateChain

example_gen_chain = QAGenerateChain.from_llm(ChatOpenAI(model=llm_model))
new_examples = example_gen_chain.apply_and_parse(
    [{"doc": t} for t in data[:5]]
)
```

将这些问答的问题传给LLM，并用debug的方式观察整个回答问题的过程，包括上面的检索过程

这个过程会打印大量中间的过程，包括：生成的prompt/token的使用量/cost/答案
```python
import langchain

langchain.debug = True
qa.run(examples[0]["query"])
```

再用LLM去给这些问答的打分，这种评估是比较难的，因为实际答案和期望答案在文字上可能完全不同的

```python
graded_outputs = eval_chain.evaluate(examples, predictions)
```

### Agent

智能体

需要一些其他工具协助LLM，回答问题，比如 计算器 or 维基百科网站

当问它一个专门领域的问题，它会先根据问题选择合适的工具，再利用工具的接口，查询出结果返回给LLM，让LLM整理输出最终结果

```python
tools = load_tools(["llm-math","wikipedia"], llm=llm)
agent= initialize_agent(
    tools, 
    llm, 
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    handle_parsing_errors=True,
    verbose = True)
agent("What is the 25% of 300?")
question = "Tom M. Mitchell is an American computer scientist \
and the Founders University Professor at Carnegie Mellon University (CMU)\
what book did he write?"
result = agent(question)
```

另外一个例子，是代码的 copilot

在LangChain中，PythonREPL作为一个工具，允许我们执行任意Python代码

```python
agent = create_python_agent(
    llm,
    tool=PythonREPLTool(),
    verbose=True
)
```
用Langchain的debug模式，可以看到更多的处理细节（我在试的时候发现一直有些错误的循环打印，不过最后的打印是对的）




## chat with your data
[chat with your data](https://learn.deeplearning.ai/courses/langchain-chat-with-your-data/lesson/snupv/introduction)

