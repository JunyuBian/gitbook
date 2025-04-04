# 关于llamaindex中LLMRerank的用法

关于今天提出的用LLM作为Reranker，从而通过prompt engineering控制rerank结果的想法，我做了如下尝试：

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

```python
template = (
    "A list of documents is shown below. Each document has a number next to it along with a summary of the document. A question is also provided. \nRespond with the numbers of the documents you should consult to answer the question, in order of relevance, as well \nas the relevance score. The relevance score is a number from 1-10 based on how relevant you think the document is to the question.\nDo not include any documents that are not relevant to the question. \nExample format: \nDocument 1:\n<summary of document 1>\n\nDocument 2:\n<summary of document 2>\n\n...\n\nDocument 10:\n<summary of document 10>\n\nQuestion: <question>\nAnswer:\nDoc: 9, Relevance: 7\nDoc: 3, Relevance: 4\nDoc: 7, Relevance: 3\n\nLet's try this now: \n\n{context_str}\nQuestion: {query_str}\nAnswer:\nYou MUST also return the number of nodes as indicated."
)
prompt_template = PromptTemplate(template)

reranker = LLMRerank(
    llm=Ollama(model="deepseek-r1:1.5b", request_timeout=30000.0),
    choice_batch_size=5,
    top_n=2,
    choice_select_prompt=prompt_template,
)

reranker.postprocess_nodes(retrieved_nodes, query_bundle)
```
