# 关于LlamaIndex里一个名字很唬人的NodeParser-SentenceWindowNodeParser

这个NodeParser和传统理解的Window没有太大关系，作用其实是在进行parse的时候，带着一部分上下句的背景，叫做SentenceContextNodeParser更为合适。

NodeParser里的Node，是文档的基本单元，可能是一个句子、段落或词语。

网上看到的一个\[示例]\(https://blog.csdn.net/xycxycooo/article/details/141813794)，可以比较好地解释这个parser的功能：

```python
# 示例文档
document = Document(text="这是一个示例文档。它包含多个句子。每个句子都将被拆分成一个节点。")

# 创建 SentenceWindowNodeParser 实例
node_parser = SentenceWindowNodeParser.from_defaults()

# 解析文档
nodes = node_parser.get_nodes_from_documents([document])

# 输出节点信息
for node in nodes:
    print(f"Node ID: {node.id_}")
    print(f"Original Text: {node.metadata[node_parser.original_text_metadata_key]}")
    print(f"Window: {node.metadata[node_parser.window_metadata_key]}")
    print("-" * 40)
```

这个示例首先创建了一个包含多个句子的文档，然后使用`SentenceWindowNodeParser`将其拆分成节点，并输出每个节点的原始文本和窗口信息。

输出结果 假设文档被拆分成以下三个句子：

“这是一个示例文档。” “它包含多个句子。” “每个句子都将被拆分成一个节点。”

那么，输出的节点信息可能如下：

```bash
Node ID: 1
Original Text: 这是一个示例文档。
Window: 这是一个示例文档。 它包含多个句子。
----------------------------------------
Node ID: 2
Original Text: 它包含多个句子。
Window: 这是一个示例文档。 它包含多个句子。 每个句子都将被拆分成一个节点。
----------------------------------------
Node ID: 3
Original Text: 每个句子都将被拆分成一个节点。
Window: 它包含多个句子。 每个句子都将被拆分成一个节点。
----------------------------------------
```

