---
layout: post
title: 通过http代理服务器连接ssh && 通过ssh上传文件
description: none
category: linux
---

今天要将下载的数据集上传到服务器，但服务器没有连外网，需要通过一个http代理服务器才可以连接上，而ssh是不能直接设置代理服务器的，必须通过tunnel。

>>> 以下是在ubuntu上完成。

+ 通过http代理服务器连接ssh

    1. 找个tunnel，比如corkscrew。
    
        $sudo apt-get install corkscrew
    
    2. 编辑ssh配置文件。将下面代码写入到~/.ssh/config

        Host *
        ProxyCommand corkscrew proxy_server proxy_port %h %p
    
    3. 可以直接使用ssh了。

        $ssh username@server


+ 通过ssh上传文件到服务器
   scp
