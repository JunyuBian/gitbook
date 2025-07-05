# Linux Patch/Git Patch的用法

patch是一个挺实用的工具，需要在明确清楚自己在干什么的基础上使用，记录一下Linux系统级别patch和git patch的区别：

## Linux Patch

### 单个文件

生成补丁

```bash
diff -u file1 file2 > diff.patch
```

打补丁

```bash
patch < diff.patch
```

### 完整目录

生成补丁

```bash
diff -urN dir1 dir2 > diff.patch
```

打补丁

```bash
patch -p1 < diff.patch
```





