---
layout: post
title: Centos 6 改 Zsh
categories: Blogs
description: Centos 6 改 Zsh
keywords: spark,yarn
---

由于长时间在实验室的虚拟机中 SSH 登录各种机器，所以今天有时间便把这台机器也改成了 zsh。下面是最终图：

![zsh](/images/posts/blog/zsh/WX20191206-231254.png)

## 1. 安装 zsh 包

yum -y install zsh

## 2. 切换默认 shell

chsh -s /bin/zsh

## 3. 重新连接 ssh

重连后会发现默认已经变成 ssh 了，可以用 echo $SHELL 查看。

## 4. 安装 oh-my-zsh

首先要安装版本高于 1.8.5 的 git，我的用 yum 安装后版本是 1.7。最后卸了，编译安装的。

1. 卸载 git

	yum remove git -y

2. 安装需要的依赖

	yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker -y

3. 下载 git 并编译。下载比较慢，我手动下下来传上去的

	``` shell
	wget https://www.kernel.org/pub/software/scm/git/git-2.11.0.tar.gz --no-check-certificate

	tar zxvf git-2.11.0.tar.gz && cd git-2.11.0

	make prefix=/usr/local/git all && make prefix=/usr/local/git install

	echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc && source /etc/bashrc
	```

4. 下载 https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh 并执行

## 5. 安装自动补全和高亮插件

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM}/plugins/zsh-syntax-highlighting

## 6. 修改主题中的文字

vim ~/.oh-my-zsh/themes/agnoster.zsh-theme

修改：

![修改主题](/images/posts/blog/zsh/WX20191206-233038.png)

## 5. 修改主题为 agnoster，添加插件

vim ~/.zshrc 搜索 ZSH_THEME="robbyrussell" robbyrussell 改成 agnoster 就可以了。

最后执行 source ~/.zshrc

完成！