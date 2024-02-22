---
layout: post
title: Mac OS 终端走 Shadowsocks 代理
date: 2018-07-31 11:19:34
subtitle: terminal_ss_proxy
categories: 其他
---

Shadowsocks：

1. 打开 SS 
2. 打开全局代理
3. 确保能科学上网

---
1. 打开 shell 配置文件
    - zsh
        ``` 
        $ open ~/.zshrc
        ```
    - bash
        ```
        $ open ~/.bashrc
        ```
2. 添加配置代码并保存, 将下面的 `1080` 替换为你的 Socks5 监听端口
    ```
    # proxy list
    alias proxy='export all_proxy=socks5://127.0.0.1:1080'
    alias unproxy='unset all_proxy'
    ```
    ![](https://ws1.sinaimg.cn/large/b92f96b9gy1ft89nxjymtj20dc0b83zc.jpg)
3. 退出终端重新进入 或者 输入命令:
    ```
    $ source ~/.zshrc
    ```
4. 验证是否成功，终端输入：
    ```
    $ proxy && curl ip.cn
    ```
    输出：
    ```
    当前 IP：35.201.114.70 来自：台湾省 Google
    ```
5. over

