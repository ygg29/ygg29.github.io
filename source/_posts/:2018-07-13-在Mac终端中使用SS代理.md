---
title: 在Mac终端中使用SS代理
date: 2018-07-13 11:21:33
category: 软件开发
tags: VPN
---

[SS](https://github.com/shadowsocks)是当前最好用的科学上网工具之一，但是在terminal中无法直接使用ss代理，需要使用额外的工具来实现这一功能。

## 准备

在开始之前，请确保：

- 拥有一个可以正常使用的SS
- Mac上已安装有[Homebrew](https://brew.sh/)

## 安装privoxy

使用以下命令安装：

```shell
brew install privoxy
```

## privoxy配置

打开配置文件，文件路径为`/usr/local/etc/privoxy/config`
在配置文件底部加入以下语句：

```shell
listen-address 0.0.0.0:8118
forward-socks5 / localhost:1080 .
```

## 启动privoxy

输入以下语句：

```shell
sudo /usr/local/sbin/privoxy /usr/local/etc/privoxy/config
```

查看是否启动成功：

```shell
netstat -na | grep 8118
```

看到如下消息说明启动成功：

```shell
tcp4 0 0 *.8118 *.* LISTEN
```

## 使用privoxy

在命令行终端中输入如下命令后，该终端即可走代理。

```shell
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
```

这时输入`curl ip.gs`会显示代理服务器的信息
关闭终端代理：

```shell
unset http_proxy
unset https_proxy
```

## 配置bash文件

经过上面几步，已经可以在终端使用SS代理，每次使用都需要输入一大串命令。可以在 ~/.bash_profile 里加入函数，便于使用。

- macOS系统中默认没有`.bash_profile`文件，需要手动创建文件：

  ```shell
  cd ~/
  touch .bash_profile
  ```

- 编辑`.bash_profile`文件：`open -e .bash_profile`

- 在`.bash_profile`里写入函数

  ```shell
  function proxy_off(){
      unset http_proxy
      unset https_proxy
      echo -e "已关闭代理"
  }
  function proxy_on() {
      export http_proxy='http://localhost:8118'
      export https_proxy='http://localhost:8118'
      echo -e "已开启代理"
  }
  ```

- 使配置立即生效：`source ~/.bash_profile`

  ## 调用bash函数

  配置已经全部完成，在终端使用`proxy_on`/`proxy_off`指令即可打开/关闭代理