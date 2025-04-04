# 关于ollama代理配置的问题

在ubuntu机器上使用ollama时，遇到了如下报错：

```bash
(edgecraftrag) labuser@labuser:~/junyu/GenAIExamples$ ollama run deepseek-r1:1.5b
pulling manifest 
Error: pull model manifest: Get "https://registry.ollama.ai/v2/library/deepseek-r1/manifests/1.5b": dial tcp 104.21.75.227:443: i/o timeout
```

问题定位在ollama没有配置代理，尝试通过`export http_proxy=<proxy address>` 方式配置，但是没有生效。

在[ollama issue](https://github.com/ollama/ollama/issues/729#issuecomment-1906311485)里找到了正确的解决方式，通过类似给docker配置代理的方式，给ollama配置代理：

```bash
sudo nano /etc/systemd/system/ollama.service
```

```bash
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

Environment="https_proxy=http://mycorporateproxy.local:8080"  #                             <---------------- 

[Install]
WantedBy=default.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama.service
```
