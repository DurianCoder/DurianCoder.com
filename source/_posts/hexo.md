---
title: hexo-github搭建个人博客
date: 2018-03-28 22:31:30
tags:
	- hexo
	- 前端
---

![](/西湖_2.jpeg)

hmmm...

来杭州实习将近三个月了...

月底返校，

闲聊之际，

使用Hexo-theme-yilia（膜拜鹅厂大佬Litten...）搭建了个人静态博客。

## 0x01、环境准备

- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/en/)

```
$ git --version
git version 2.11.1.windows.1
$ npm -v
5.6.0
$ node -v
v8.10.0
```

如果您电脑已经安装以上环境，接下来只需使用npm即可完成Hexo的安装：

```
# 安装hexo
$ npm install -g hexo-cli 
# 检查hexo版本,如显示版本信息，则安装成功
$ hexo -v
```



## 0x02、Hexo建站

安装Hexo完成后，初始化Hexo:

```
# 进入到安装目录
$ cd <folder>
# 初始化Hexo，出现"Start blogging with Hexo"则初始化完成
$ hexo init 
# npm安装所需组件
$ npm install
```

启动服务器，访问http://localhost:4000/：

```
$ hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.

```

​	Tips：如果一直无法跳转，可能端口被占用，ctrl + c 停止服务更改端口：`hexo server -p port_num`

至此Hexo本地搭建完成.



## 0x03、将Hexo与Github连接起来

> 本地配置git：

```
# 配置git用户名和邮箱
$ git config --global user.name "your_github_name"
$ git config --global user.email "your_github_email"
$ cd ~/.ssh
# 生成密钥
$ ssh-keygen -t rsa -C "your_github_email"
# 将密钥添加到ssh-agent
$ eval"(ssh-agent -s)"
# 将生成的SSH key添加到ssh-agent
$ ssh-add~/.ssh/id_rsa
```

> 配置github密钥，将`id_rsa.pub`添加到`Key`。

> 测试ssh是否添加成功：

```
ssh -T git@github.com
```

> 部署：

配置hexo根目录下的_config.yml文件

```
deploy:
	type: git
	repository: "Github的Project路径，Project name 必须为 github_name.github.io"
	branch: 分支
```



## 0x04、新建博客

新建博客-->编辑md文件-->博客生成、部署

```
$ hexo new "hexo"
INFO  Created: E:\Hexo\source\_posts\hexo.md
$ hexo d -g
```

到此hexo静态博客搭建完成，访问`githubname.github.io`博客链接。

#### 参考文档：

​	[Hexo](https://hexo.io/docs/)

​	[Hexo-主题](https://hexo.io/zh-cn/docs/index.html)

​	[使用Hexo+Github一步步搭建属于自己的博客（基础）](http://www.cnblogs.com/fengxiongZz/p/7707219.html)