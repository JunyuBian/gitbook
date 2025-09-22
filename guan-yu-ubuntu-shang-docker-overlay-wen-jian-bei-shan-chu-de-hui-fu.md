# 关于ubuntu上docker overlay文件被删除的恢复

q前两天在ubuntu上清理空间时，把docker的overlay文件删除了，之后一直报错：

```bash
rror response from daemon: open /var/lib/docker/overlay2/blah/blah/blah/dir_000027_000001.xhtml: no such file or directory
```

看到[讨论](https://app.gitbook.com/u/z7HMqniEKrQfjlbddT9XtmPZ8fi2)后，通过下面两个步骤解决：

```bash
docker system prune -af
```

运行成功后，再运行：

```bash
sudo service restart docker
```

fixed the issue...



