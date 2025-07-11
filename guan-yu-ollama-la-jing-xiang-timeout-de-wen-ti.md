# 关于Ollama拉镜像timeout的问题

在公司内网运行ollama，在拉取镜像时，会timeout报错：

![](.gitbook/assets/image.png)

尝试配置代理后运行，依旧存在相同报错。

最后按照[论坛](https://github.com/ollama/ollama/issues/3816)里的说法，解决了这个问题：

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>







