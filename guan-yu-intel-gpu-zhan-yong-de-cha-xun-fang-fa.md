# 关于Intel GPU占用的查询方法

在测试deepseek-distill-qwen-32B的显存占用问题时，需要一个工具来查看Arc A770的占用情况，找到了一个内容全面、界面友好的工具：nvtop。

## Github网址

[https://github.com/Syllo/nvtop?tab=readme-ov-file#nvtop](https://github.com/Syllo/nvtop?tab=readme-ov-file#nvtop)

## 注意事项

大部分使用方法都在github page上写出来了，需要注意的一点是，如果想要查看Intel GPU的占用情况，需要自己build nvtop，而不是使用发行版。

build方式参考：[https://github.com/Syllo/nvtop?tab=readme-ov-file#nvtop](https://github.com/Syllo/nvtop?tab=readme-ov-file#nvtop)

在build时需要设置：`-DINTEL_SUPPORT=ON`

build结束后，需要在build路径中，找到可执行的nvtop文件运行。

## 其他选择

最近接触到了一个新的命令行工具：xpu-smi dump &#x20;

使用样例：

```
xpu-smi dump -m 5

Timestamp, DeviceId, GPU Memory Utilization (%)
15:39:54.882,    0, 99.37
15:39:54.882,    1, 85.17
15:39:54.882,    2, 85.51
15:39:54.882,    3, 85.17
15:39:55.882,    0, 99.37
15:39:55.882,    1, 85.17
15:39:55.882,    2, 85.51
15:39:55.882,    3, 85.17
15:39:56.882,    0, 99.37
15:39:56.882,    1, 85.17
15:39:56.882,    2, 85.51
15:39:56.882,    3, 85.17 
```
