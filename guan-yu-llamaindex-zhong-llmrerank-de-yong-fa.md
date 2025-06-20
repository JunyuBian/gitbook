# 关于llamaindex中LLMRerank的用法

## 工程实现

关于今天提出的用LLM作为Reranker，从而通过prompt engineering控制rerank结果的想法，我基于llamaindex的[LLMRerank](https://docs.llamaindex.ai/en/stable/api_reference/postprocessor/llm_rerank/)做了如下尝试：

前面几步比较常规，首先是通过SimpleDirectoryReader加载文档：

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.postprocessor import LLMRerank

# load documents
documents = SimpleDirectoryReader("/home/labuser/junyu/GenAIExamples/EdgeCraftRAG/test_docs/non-clean").load_data()
```

接着通过VectorStoreIndex获取index，这里有一个坑，和Yongbo在EC-RAG里的发现一致，需要把`Settings.embed_model`设置为`None` ，否则会报没有OpenAI Key的错：

```python
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core import QueryBundle

# Settings.embed_model should be set to None when embed_model is None to avoid 'no oneapi key' error
from llama_index.core import Settings

Settings.embed_model = None

index = VectorStoreIndex.from_documents(
    documents,
)
```

接着配置retriever：

```python
query_bundle = QueryBundle("how to trouble shoot flux")
# configure retriever
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=30,
)
retrieved_nodes = retriever.retrieve(query_bundle)
```

最后是重点的配置LLMRerank的部分，llamaindex给出了一个[示例](https://docs.llamaindex.ai/en/stable/examples/node_postprocessor/LLMReranker-Gatsby/)，使用的OpenAI的model，不符合我们的使用情况，如果希望用自定义的llm，官方的这个[示例](https://docs.llamaindex.ai/en/stable/module_guides/models/llms/usage_custom/)，同时结合llamaindex[支持的modules列表](https://docs.llamaindex.ai/en/stable/module_guides/models/llms/modules/)，就可以通过下面的代码使用自定义的llm了，比如这里跑在ollama上的deepseek-r1:1.5b：

<pre class="language-python"><code class="lang-python">from llama_index.core import PromptTemplate

template = (
    "A list of documents is shown below. Each document has a number next to it along with a summary of the document. A question is also provided. \nRespond with the numbers of the documents you should consult to answer the question, in order of relevance, as well \nas the relevance score. The relevance score is a number from 1-10 based on how relevant you think the document is to the question.\nDo not include any documents that are not relevant to the question. \nExample format: \nDocument 1:\n&#x3C;summary of document 1>\n\nDocument 2:\n&#x3C;summary of document 2>\n\n...\n\nDocument 10:\n&#x3C;summary of document 10>\n\nQuestion: &#x3C;question>\nAnswer:\nDoc: 9, Relevance: 7\nDoc: 3, Relevance: 4\nDoc: 7, Relevance: 3\n\nLet's try this now: \n\n{context_str}\nQuestion: {query_str}\nAnswer:\nYou MUST also return the number of nodes as indicated."
)
prompt_template = PromptTemplate(template)


from llama_index.llms.ollama import Ollama

reranker = LLMRerank(
    llm=Ollama(model="deepseek-r1:1.5b", request_timeout=30000.0),
    choice_batch_size=5,
    top_n=2,
    choice_select_prompt=prompt_template,
)

<strong>reranker.postprocess_nodes(retrieved_nodes, query_bundle)
</strong></code></pre>

## Llmaindex LLMReranker源码

源码看起来比较直观，直接贴在最下面了，主要有一个参数比较有意思：`choice_batch_size`&#x20;

下面是GPT老师的解析：

在这段代码中，批处理是通过将节点列表分成多个小批次来进行的。批处理的目的是提高处理效率，尤其是在处理大量数据时。以下是代码中批处理的详细解释：

#### 批处理的实现

1. **批次大小**:
   * `self.choice_batch_size` 是一个属性，定义了每个批次处理的节点数量。
2. **分批处理节点**:
   * 使用 `range(0, len(nodes), self.choice_batch_size)` 创建一个循环，步长为 `self.choice_batch_size`。这意味着每次循环处理 `self.choice_batch_size` 个节点。
   * `idx` 是当前批次的起始索引。
3. **提取当前批次的节点**:
   * `nodes_batch = [node.node for node in nodes[idx : idx + self.choice_batch_size]]` 这行代码从 `nodes` 列表中提取当前批次的节点。
   * `nodes[idx : idx + self.choice_batch_size]` 使用切片操作获取当前批次的节点。
   * `[node.node for node in ...]` 提取每个节点的 `node` 属性，形成 `nodes_batch` 列表。
4. **处理每个批次**:
   * 对于每个 `nodes_batch`，执行以下操作：
     * **格式化批次字符串**: 使用 `self._format_node_batch_fn(nodes_batch)` 方法对批次进行格式化。
     * **预测**: 使用 `self.llm.predict` 方法进行预测，传入格式化的批次字符串和查询字符串。
     * **解析结果**: 使用 `self._parse_choice_select_answer_fn` 方法解析预测结果，得到选择的节点索引和相关性分数。
     * **选择节点**: 根据解析结果选择节点，并为每个节点分配相关性分数。
5. **扩展结果列表**:
   * 将选择的节点和相关性分数添加到 `initial_results` 列表中。

关于选择节点：

在这段代码中，选择节点的功能是通过解析语言模型的预测结果来实现的。具体来说，代码使用语言模型对每个批次的节点进行预测，然后根据预测结果选择最相关的节点。以下是选择节点功能的详细解释：

#### 选择节点的步骤

1. **预测结果**:
   * 使用 `self.llm.predict` 方法对当前批次的节点进行预测。预测的输入包括格式化的批次字符串和查询字符串。
   * `self.choice_select_prompt` 是用于指导预测的提示模板。
2. **解析预测结果**:
   * 使用 `self._parse_choice_select_answer_fn` 方法解析预测结果。这个方法的作用是从语言模型的输出中提取选择的节点索引和相关性分数。
   * `raw_response` 是语言模型的原始输出。
   * `raw_choices` 是解析后得到的节点选择索引列表。
   * `relevances` 是解析后得到的节点相关性分数列表。
3. **选择节点**:
   * `choice_idxs = [int(choice) - 1 for choice in raw_choices]` 将选择的节点索引转换为整数，并调整为从零开始的索引。
   * `choice_nodes = [nodes_batch[idx] for idx in choice_idxs]` 根据选择的索引从当前批次中提取对应的节点。
   * `relevances = relevances or [1.0 for _ in choice_nodes]` 如果没有相关性分数，则默认分配一个分数（例如1.0）。
4. **构建结果列表**:
   * `initial_results.extend([...])` 将选择的节点和相关性分数添加到 `initial_results` 列表中。
   * 每个节点被包装成 `NodeWithScore` 对象，其中包含节点和其相关性分数。

简单来说，llamaindex在实现的过程中，把拿到的带有score的node，分批次打分，并append结果列表里，循环打分结束，返回列表里得分比较高的node。

这样会导致一些问题，比如，batch size不同，大模型可能会产生不同的打分；比如，node出现的位置、顺序不同，可能会出现不同的打分等。对于这些问题的处理有待提升。

源码如下(`llama-index-core/llama_index/core/postprocessor/llm_rerank.py`)：

```python
class LLMRerank(BaseNodePostprocessor):
    """LLM-based reranker."""

    top_n: int = Field(description="Top N nodes to return.")
    choice_select_prompt: SerializeAsAny[BasePromptTemplate] = Field(
        description="Choice select prompt."
    )
    choice_batch_size: int = Field(description="Batch size for choice select.")
    llm: LLM = Field(description="The LLM to rerank with.")

    _format_node_batch_fn: Callable = PrivateAttr()
    _parse_choice_select_answer_fn: Callable = PrivateAttr()

    def __init__(
        self,
        llm: Optional[LLM] = None,
        choice_select_prompt: Optional[BasePromptTemplate] = None,
        choice_batch_size: int = 10,
        format_node_batch_fn: Optional[Callable] = None,
        parse_choice_select_answer_fn: Optional[Callable] = None,
        top_n: int = 10,
    ) -> None:
        choice_select_prompt = choice_select_prompt or DEFAULT_CHOICE_SELECT_PROMPT

        llm = llm or Settings.llm

        super().__init__(
            llm=llm,
            choice_select_prompt=choice_select_prompt,
            choice_batch_size=choice_batch_size,
            top_n=top_n,
        )
        self._format_node_batch_fn = (
            format_node_batch_fn or default_format_node_batch_fn
        )
        self._parse_choice_select_answer_fn = (
            parse_choice_select_answer_fn or default_parse_choice_select_answer_fn
        )

    def _get_prompts(self) -> PromptDictType:
        """Get prompts."""
        return {"choice_select_prompt": self.choice_select_prompt}

    def _update_prompts(self, prompts: PromptDictType) -> None:
        """Update prompts."""
        if "choice_select_prompt" in prompts:
            self.choice_select_prompt = prompts["choice_select_prompt"]

    @classmethod
    def class_name(cls) -> str:
        return "LLMRerank"

    def _postprocess_nodes(
        self,
        nodes: List[NodeWithScore],
        query_bundle: Optional[QueryBundle] = None,
    ) -> List[NodeWithScore]:
        if query_bundle is None:
            raise ValueError("Query bundle must be provided.")
        if len(nodes) == 0:
            return []

        initial_results: List[NodeWithScore] = []
        for idx in range(0, len(nodes), self.choice_batch_size):
            nodes_batch = [
                node.node for node in nodes[idx : idx + self.choice_batch_size]
            ]

            query_str = query_bundle.query_str
            fmt_batch_str = self._format_node_batch_fn(nodes_batch)
            # call each batch independently
            raw_response = self.llm.predict(
                self.choice_select_prompt,
                context_str=fmt_batch_str,
                query_str=query_str,
            )

            raw_choices, relevances = self._parse_choice_select_answer_fn(
                raw_response, len(nodes_batch)
            )
            choice_idxs = [int(choice) - 1 for choice in raw_choices]
            choice_nodes = [nodes_batch[idx] for idx in choice_idxs]
            relevances = relevances or [1.0 for _ in choice_nodes]
            initial_results.extend(
                [
                    NodeWithScore(node=node, score=relevance)
                    for node, relevance in zip(choice_nodes, relevances)
                ]
            )

        return sorted(initial_results, key=lambda x: x.score or 0.0, reverse=True)[
            : self.top_n
        ]
```
