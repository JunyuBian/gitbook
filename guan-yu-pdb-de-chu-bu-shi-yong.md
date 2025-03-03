# 关于pdb的初步使用

\[pdb]\([https://docs.python.org/zh-cn/3.13/library/pdb.html](https://docs.python.org/zh-cn/3.13/library/pdb.html))可以用于分步骤执行、调试python程序。

使用的时候需要在程序中插入：

```python
import pdb; pdb.set_trace()
```

调试器的提示符为 `(Pdb)`，这指明你正处于调试模式下:

```bash
(Pdb) p x
3
(Pdb) continue
3 * 2 is 6
```

具体使用的命令可以参考\[这篇总结]\([https://blog.csdn.net/qq\_43799400/article/details/122582895](https://blog.csdn.net/qq_43799400/article/details/122582895))，比较全面。

看到网上还有另一种用法：

```python
python -m pdb myscript.py
```

没有用过，有时间尝试一下。
