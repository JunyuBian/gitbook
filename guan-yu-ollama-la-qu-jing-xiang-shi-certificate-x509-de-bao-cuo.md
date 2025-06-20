# 关于ollama拉取镜像时certificate x509的报错

今天在测试LLMRerank接入ollama的后端llm，准备运行BAAI/bge-reranker-v2-m3作为reranker的模型，但是运行`ollama run BAAI/bge-reranker-v2-m3` 时，出现了如下报错：

```bash
Error: pull model manifest: Get "https://registry.ollama.ai/v2/BAAI/bge-reranker-v2-m3/manifests/latest": tls: failed to verify certificate: x509: certificate signed by unknown authority
```

网络上有一些[相关讨论](https://github.com/ollama/ollama/issues/823)，但是是针对在docker里启动ollama的情况，可以参考，但对于当前case意义不大。

报错的根源在于，相关模型在ollama里的命名，和huggingface不一致。

比如BAAI/bge-reranker-v2-m3，在huggingface里名称如下：

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

但是在ollama中需要选择其他名称，比如：

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
