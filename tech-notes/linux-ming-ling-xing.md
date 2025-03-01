# Linux命令行

## \[jq] 命令行处理JSON

e.g.

```bash
$ echo '{ "foo": { "bar": { "baz": 123 } } }' | jq '.'
{
  "foo": {
    "bar": {
      "baz": 123
    }
  }
}
```

## \[[nvtop](https://github.com/Syllo/nvtop)] monitor for GPUs and accelerators

对于Intel GPU的支持，需要从代码库build本地nvtop，同时`sudo setcap cap_perfmon=ep nvtop`

build之前，需要确保依赖已经安装：`sudo apt install cmake libncurses5-dev libncursesw5-dev git`

## \[[optimum-cli](https://github.com/huggingface/optimum-intel)] 转换huggingface的模型

转为openVINO并量化：`optimum-cli export openvino -m deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B ${MODEL_PATH}/deepseek-r1-distill-qwen-1.5b-openvino --weight-format int4`

* \[docker] 配置代理

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/proxy.conf
```

在这个proxy.conf文件（可以是任意\*.conf的形式）中，添加以下内容：

```bash
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

重启生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## \[docker] 取消sudo操作

创建docker组：`sudo groupadd docker`

添加当前用户到docker组：`sudo gpasswd -a ${USER} docker`

重启docker服务：`sudo service docker restart`

刷新docker成员：`newgrp - docker`

## \[conda] 配置代理

```bash
vim ~/.condarc
```

添加：

```bash
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
 
proxy_servers:
  http: http://xxx.xx.com:8080
  https: https://xxx.xx.com:8080
ssl_verify: false
```
