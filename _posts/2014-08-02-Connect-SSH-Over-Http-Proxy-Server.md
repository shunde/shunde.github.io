---
layout: post
title: 通过 http 代理链接 ssh 
category: Linux
---

今天要将下载的数据集上传到服务器，但服务器没有连外网，需要通过一个 http 代理服务器才可以连接上，而 ssh 是不能直接设置代理服务器的，必须通过 tunnel。

> 以下是在 ubuntu 上完成。

### 通过 http 代理服务器连接 ssh

1. 找个tunnel，比如corkscrew。
    
    $sudo apt-get install corkscrew
    
2. 编辑ssh配置文件。将下面代码写入到~/.ssh/config

    Host *
    ProxyCommand corkscrew proxy_server proxy_port %h %p
    
3. 可以直接使用ssh了。

    $ssh username@server


-EOF-