---
title: git 操作权限不够
date: 2018-03-30 10:50:47
tags:
	- git
---

当在本地操作github远程仓库时经常出现权限不足、无法访问问题，可能是本地密钥失效。

![/error.png]()

<!--more-->

### 0x01、git clone 报错权限不足问题

- 本地公钥失效，重新配置
```
$ ssh-keygen -t rsa -C "your_github_email"
```
- 配置github密钥，将id_rsa.pub添加到Key
- 验证密钥是否配置成功
```
$ ssh -T git@github.com
```
