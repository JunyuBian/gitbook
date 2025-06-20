# 关于调试Prompt的迭代流程

### 1. 结构 <a href="#id-1-jie-gou" id="id-1-jie-gou"></a>

plain text -> 结构化分段的text -> 段落中的text分点列举

#### 1.1 plain text <a href="#id-11-plain-text" id="id-11-plain-text"></a>

用纯文本罗列prompt，包括role定义、回答格式要求等，比如：

```
You are an AI assistant. Your task is to learn from the following context. Then answer the user's question based on what you learned from the context but not your own knowledge.
When responding, please keep the following points in mind: Translate any Chinese text into English when providing answers. Not all content in the search results is closely related to the user's question. You need to evaluate and filter the search results based on the question...
```

#### 1.2 结构化分段 <a href="#id-12-jie-gou-hua-fen-duan" id="id-12-jie-gou-hua-fen-duan"></a>

用明确的结构把plain text分成几个大段落，比如：

```
### Instruction ###
You are an AI assistant. Your task is to learn from the following context and must make sure understand follow Chain of Thought. 
Then answer the user's question based on what you learned from the context but not your own knowledge. 
You must strictly follow the points listed in Chain of Thought when providing answers. For example, if there're five points in Chain of Thought, your answer must align with these five points. And each point must follow the summary provided in Chain of Thought.
### Additional Information ###
When responding and thinking, please keep the following points in mind: nozzle is pick up die component to pick die and place it on substrate for bonding, Theta is the parameter value for the angle of the nozzle when the BH picks from the OT and BS nozzle bank, which is then d by the OT uplook camera, Calculate similarity and offset by comparing the image gnition system with the database.
```

#### 1.3 段落中的text分点列举 <a href="#id-13-duan-luo-zhong-de-text-fen-dian-lie-ju" id="id-13-duan-luo-zhong-de-text-fen-dian-lie-ju"></a>

在段落中，把每个点用数字或者破折号列举出来，比如：

```
### Procedures ###
The follow list is how you should consider the answer, you must follow these steps when responding:
1. Validate the tool and product configuration
2. Perform the height of the pickup nozzle calibration
3. false inspection result of nozzle theta
4. Validate the inspection result <True/False> and  stability of nozzle theta
5. nozzle theta unstable, need to adjust
### Additional Information ###
When responding and thinking, please keep the following points in mind:
- nozzle is pick up die component to pick die and place it on substrate for bonding
- Theta is the parameter value for the angle of the nozzle when the BH picks from the OT and BS nozzle bank, which is then d by the OT uplook camera
- If the similarity is below the set threshold, it will trigger an alarm for the downlook PR issue.
- You must translate any Chinese text into English in answer and thinking process, the user will not understand Chinese.
```

### 2. 语气 <a href="#id-2-yu-qi" id="id-2-yu-qi"></a>

常规对话语气 -> 命令式语气 -> 命令式语气+大写must/SHOULD

#### 2.1 常规对话语气 <a href="#id-21-chang-gui-dui-hua-yu-qi" id="id-21-chang-gui-dui-hua-yu-qi"></a>

用平常的、非强制的语气写prompt，如：

```
When responding, please keep the following points in mind:
```

#### 2.2 命令式语气 <a href="#id-22-ming-ling-shi-yu-qi" id="id-22-ming-ling-shi-yu-qi"></a>

在句子中加入语气变化，主要是变成下达命令的语气，比如：

```
You must translate any Chinese text into English in answer and thinking process, the user will not understand Chinese. English must be guaranteed.
```

#### 2.3 命令式语气基础上增加大写 <a href="#id-23-ming-ling-shi-yu-qi-ji-chu-shang-zeng-jia-da-xie" id="id-23-ming-ling-shi-yu-qi-ji-chu-shang-zeng-jia-da-xie"></a>

强化命令，增加更加绝对的词语，并且把他们大写，比如：

```
You MUST translate any Chinese text into English in answer and thinking process, the user WILL NOT understand Chinese. English MUST be guaranteed.
```

### 3. 内容 <a href="#id-3-nei-rong" id="id-3-nei-rong"></a>

限制回答点数、回答格式 -> 放宽点数限制，转向限制回答内容（命令式要求和COT相符合） -> 点数限制基础上，进行举例

#### 3.1 限制回答点数、回答格式 <a href="#id-31-xian-zhi-hui-da-dian-shu-hui-da-ge-shi" id="id-31-xian-zhi-hui-da-dian-shu-hui-da-ge-shi"></a>

在prompt里对大模型回答的点数、格式进行hardcode的限制，比如：

```
If the response is lengthy, structure it well and summarize it in paragraphs. If a point-by-point format is needed, try to limit it to 5 points and merge related content.
```

#### 3.2 放宽点数限制 <a href="#id-32-fang-kuan-dian-shu-xian-zhi" id="id-32-fang-kuan-dian-shu-xian-zhi"></a>

主要是不再hardcore限制回答数量，转为限制大模型的回答必须和COT相符，比如：

```
You MUST STRICTLY follow the points listed in Procedures when providing answers. 
```

#### 3.3 在点数限制基础上，进行举例 <a href="#id-33-zai-dian-shu-xian-zhi-ji-chu-shang-jin-xing-ju-li" id="id-33-zai-dian-shu-xian-zhi-ji-chu-shang-jin-xing-ju-li"></a>

在上面的prompt里，增加一些例子，让大模型回答的时候进行模仿，如：

```
You MUST STRICTLY follow the points listed in Procedures when providing answers. For example, if there're five points in Procedures, your answer must align with these five points. And each point MUST follow the summary provided in Procedures.
```
