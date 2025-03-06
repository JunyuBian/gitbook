# RAG Parser调查结果

parser的实验结果我放在下面的表里了，主要是测试了\[文章]\([https://arxiv.org/abs/2410.09871](https://arxiv.org/abs/2410.09871))里（涵盖了我能找到的所有主流parser）的7种传统parser+2种有大模型辅助的parser。表格里面包括，调用parser的效果和对parser使用的总结，对比结果看：

1. 除了tabula只能处理表格，不符合使用场景，7种传统parser表现差别不大
2. 两种大模型辅助的parser，我觉得不太符合RAG的应用场景：
   1. facebook的nougat：主要功能为自动解析论文PDF，输出生成可编辑的Markdown+LaTex代码，用在pdf论文编辑上多一些
   2. microsoft的table-transformer：主要通过物体检测，识别图片、pdf文件里的表格，优势在于可以识别图片里的表格，但是需要和额外的OCR/文本检测工具配合来提取文字
3. 在7种传统parser里有三种parser有一些优势：
   1. pdfminer.six：主要优势在于，处理后原生输出的内容里不包含换行符等特殊字符
   2. PyMuPDF：主要优势在于，可以提取文档目录（doc.get\_toc()），且比较整洁，比如：\[1, 'Chapter 1          Safety Instructions', 9], \[2, '1.1 Safety Precautions', 9], \[2, '1.2  List of Safety Remarks', 10], \[2, '1.3 List of Warning Labels on Machine', 14], \[3, '1.3.1 Labels on PHX-Output (Output Shuttle)', 14], \[3, '1.3.2 Labels on PHX-Input (Input Shuttle)', 16]
   3. PyPDF2： 主要优势在于，也可以提取文档目录（doc.outline），但输出更复杂一些，比如：{'/Title': 'Chapter 1          Safety Instructions', '/Page': IndirectObject(25, 0, 127814997869424), '/Type': '/XYZ', '/Left': 54, '/Top': 768, '/Zoom': 0, '/Count': -17}, \[{'/Title': '1.1 Safety Precautions', '/Page': IndirectObject(25, 0, 127814997869424), '/Type': '/XYZ', '/Left': 54, '/Top': 722, '/Zoom': 0}, {'/Title': '1.2  List of Safety Remarks', '/Page': IndirectObject(27, 0, 127814997869424), '/Type': '/XYZ', '/Left': 54, '/Top': 768, '/Zoom': 0}&#x20;

实验结果表：

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
