# 关于sudo运行python脚本时库文件找不到的问题

在用户态通过`pip install -r requirements.txt安装依赖后，需要使用sudo运行脚本，但是发生类似如下报错：`

```
ModuleNotFoundError: No module named 'uvicorn'
```

通过`sys.path可以看到，用户态和sudo的python库路径不同：`

```
labuser@labuser:~/junyu/GenAIExamples$ python3
Python 3.10.16 | packaged by conda-forge | (main, Dec  5 2024, 14:16:10) [GCC 13.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/home/labuser/miniforge3/envs/edgecraftrag/lib/python310.zip', '/home/labuser/miniforge3/envs/edgecraftrag/lib/python3.10', '/home/labuser/miniforge3/envs/edgecraftrag/lib/python3.10/lib-dynload', '/home/labuser/miniforge3/envs/edgecraftrag/lib/python3.10/site-packages']
```

```
labuser@labuser:~/junyu/GenAIExamples$ sudo python3
[sudo] password for labuser: 
Python 3.10.12 (main, Feb  4 2025, 14:57:36) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/usr/lib/python310.zip', '/usr/lib/python3.10', '/usr/lib/python3.10/lib-dynload', '/usr/local/lib/python3.10/dist-packages', '/usr/lib/python3/dist-packages', '/home/labuser/miniforge3/envs/edgecraftrag/lib/python3.10']
```

通过在代码里显式添加sys path后，sudo也可正常运行：

````
```python
import sys

sys.path.append("/home/labuser/miniforge3/envs/edgecraftrag/lib/python3.10/site-packages")

print("done with sys.path.append............")

```
````
