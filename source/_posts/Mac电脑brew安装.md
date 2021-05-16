---
title: Mac电脑brew安装
date: 2020-03-15 13:10:16
tags: 
    - Mac
    - brew
---
#### 前言
  国内因为网络的缘故，安装brew如龟速，而且经常中途失败，怎么快速安装就成了保持好心情的必备法宝。

#### brew 安装
最近新安装了Mac系统导致电脑环境需要重新配置，像之前一样操作：
```sh
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install >> brew_install
```
but写入文件的内容是：
```text
#!/usr/bin/ruby

STDERR.print <<~EOS
  Warning: The Ruby Homebrew installer is now deprecated and has been rewritten in
Bash. Please migrate to the following command:
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

EOS

Kernel.exec "/bin/bash", "-c", '/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"'
```
找了一会儿原因，发现是网址变了，最新的地址是:
```text
https://raw.githubusercontent.com/Homebrew/install/master/install.sh
```
修改之后，输入：
```sh
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh >> brew_install
```
打开**brew_install**文件，修改**BREW_REPO**的值：
```
# BREW_REPO="https://github.com/Homebrew/brew"
BREW_REPO="https://mirrors.aliyun.com/homebrew/brew.git"
```
保存，命令行输入：
```sh
/bin/bash brew_install
```
发现卡在了：
```text
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core'...
```
Ctrl + C 中止，输入以下命令：
```sh
mkdir -p /usr/local/Homebrew/Library/Taps/homebrew/
cd /usr/local/Homebrew/Library/Taps/homebrew
git clone https://mirrors.aliyun.com/homebrew/homebrew-core.git
```
完成之后，运行**brew help**, 显示help信息说明安装成功。
#### brew cask安装
brew源检查，分别输入：
```shell
git -C "$(brew --repo)" remote -v
git -C "$(brew --repo homebrew/core)" remote -v
```
如果显示不是阿里云的源，按照[阿里云Homebrew源](https://developer.aliyun.com/mirror/homebrew)的操作配置，成功之后输入：
```
brew install brew-cask-completion
```
中间卡在：
```
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask'...
```
Ctrl + C 中止，然后输入：
```
cd /usr/local/Homebrew/Library/Taps/homebrew
git clone https://mirrors.ustc.edu.cn/homebrew-cask.git
```
因为没有找了阿里云cask的源，只好用科大的了，输入**brew cask help**，显示help信息安装成功。
接着输入：
```sh
git -C "$(brew --repo homebrew/cask)" remote -v
```
不是国内源的话，修改成科大的源：
```
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```
以上，brew安装完成。
#### 参考内容
[阿里云](https://developer.aliyun.com/mirror/?spm=a2c6h.13651104.0.d1002.3e676e2eNAwfow)
[中国科学技术大学开源软件镜像](http://mirrors.ustc.edu.cn/)