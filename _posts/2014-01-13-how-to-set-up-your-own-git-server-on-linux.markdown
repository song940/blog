---
layout: post
title: "How to set up your own Git server on Linux"
date: 2014-01-13 14:33
comments: true
categories: linux
---

Git 作为版本控制工具无疑是软件开发最好的选择, 而 Github 又是 Git 服务中做的最好的, 但是这并不妨碍我们搭建自己的 Git Server .


##Set up the server

连接到远程服务器

```bash
ssh lsong.org
```

添加 git 用户

	sudo adduser git

配置 SSH-Key 登录

```bash
scp ~/.ssh/id_rsa.pub git@lsong.org:~/lsong-key.pub

ssh git@lsong.org
...
mkdir /home/git/.ssh
cat lsong-key.pub >> .ssh/authorized_keys
sudo chown -R git:git /home/git/.ssh
sudo chmod 700 !$
sudo chmod 600 /home/git/.ssh/*
```

现在应该可以在本地通过 	`git` 用户登录服务器了

	ssh git@lsong.org


##开始一个项目

在服务器上启动一个项目

	cd ~git/
	mkdir project.git
	cd project.git
	git --bare init

在本地开始这个项目

	cd myproject
	git init
	git add .
	git commit -m 'initial commit'
	git remote add origin git@lsong.org:project.git
	git push origin master

你也可以 `clone` 这个项目

	git clone git@lsong.org:project.git

##权限控制

###读写权限

让需要 `git` 服务写权限的用户将 SSH 公钥发给你

	cat ~/.ssh/id_rsa.pub
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
	ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
	Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
	Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
	O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
	dAv8JggJICUvax2T9va5 gsg-keypair

然后你需要

	cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys

这个用户就可以提交了

###只读权限

	cd ~git/public_html
	ln -s ../project.git project.git

要开启需要

	cd project.git
	mv hooks/post-update.sample ./hooks/post-update
	chmod a+x ./hooks/post-update
	./hooks/post-update

任何人都可以通过

	git clone https://lsong.org/~git/project.git

来 `clone` 项目

##安全性

你可以用 Git 自带的 `git-shell` 工具限制 `git` 用户的活动范围。只要把它设为 `git` 用户登入的 shell，那么该用户就无法使用普通的 bash 或者 csh 什么的 shell 程序。编辑 `/etc/passwd` 文件：

	sudo vim /etc/passwd

在文件末尾，你应该能找到类似这样的行：

	git:x:1000:1000::/home/git:/bin/sh

把 	bin/sh	 改为 	/usr/bin/git-shell	 （或者用 `which git-shell` 查看它的实际安装路径）。该行修改后的样子如下：

	git:x:1000:1000::/home/git:/usr/bin/git-shell

现在 `git` 用户只能用 SSH 连接来推送和获取 Git 仓库，而不能直接使用主机 shell。尝试普通 SSH 登录的话，会看到下面这样的拒绝信息：

	$ ssh git@gitserver
	fatal: What do you think I am? A shell?
	Connection to gitserver closed.
