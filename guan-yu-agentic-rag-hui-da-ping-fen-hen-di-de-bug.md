# 关于Agentic RAG回答评分很低的bug

在对[EC-RAG](https://github.com/opea-project/GenAIExamples/tree/main/EdgeCraftRAG)进行agentic模式开发时，遇到了一个问题，在EC-RAG基础上增加Agentic流程后，因为对回答添加了更多规划，按理说回答应该能够得到更高的得分，但是打分出来，结果却比native pipeline要低。

做了几个实验，最终在vllm、agentic、EC-RAG输出的log里看到了问题。

agentic在做完retrieval和rerank之后，会把拿到的context放到CONTEXT的role里，传递到给vllm generator的prompt里，但是vllm打印结果显示，prompt里没有CONTEXT role的内容，等于agentic一直在没有context的情况下回答问题。

这个问题的根本原因是，vllm只会接受特定role的内容，但是context关键字，不在其中，导致vllm自动过滤掉了prompt里context role内的文本。







