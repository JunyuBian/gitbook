# 关于Deepseek模型跳过思考过程的问题

部署Deepseek模型后（distilled-qwen系列），发现模型经常性跳过思考过程，回答会从\</think>开始，从deepseek的[说明](https://github.com/deepseek-ai/DeepSeek-R1?tab=readme-ov-file#usage-recommendations)来看，这个行为在官方的预期之内，但是可以通过调节prompt，强制模型进行思考。

尝试后发现下面两个方式叠加使用，可以让模型输出思考过程：

1. 在prompt里添加，明确指出需要think：`MUST initiate your response with "<think>\n Okay" at the beginning of every output.`
2. 引导模型输出以\<think>开头的答案：`ret = "User: {}\nAssistant: \n Okay, so ".format(question)`
