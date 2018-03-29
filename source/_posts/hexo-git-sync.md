---
title: hexo-git-sync
date: 2018-03-29 22:29:27
tags:
	- hexo
---

更换电脑时如何写blog..
hexo博客文档树：
![image](/tree.png)

下面介绍一下，如何将hexo博客上传到github，便于在更换电脑时推送博客。
## 0x01、在仓库新建分支``hexo``
> 将hexo分支设置为默认分支



## 0x02、将仓库克隆到本地

```
$ git clone -b hexo git@github.com:DurianCoder/DurianCoder.github.io.git
# 查看分支
$ git branch

```


## 0x03、配置本地仓库

>将本地部署的文件（hexo中的全部文件）拷贝到username.github.io文件目录

>将theme目录中主题内的.git目录删除



## 0x04、提交分支

```
$ git add .
$ git commit -m "back up hexo file"
$ git push
```
至此已经将hexo成功上传到了blog的hexo分支



## 0x05、在其他电脑上pull配置博客

> 将新电脑的生成的ssh key添加到GitHub账户上

> 克隆username.github.io的xxx分支

> 切换到username.github.io目录，执行npm install



## 0x06、写博客

```
# 先更新博客
$ git pull

# 新建博客、写博客
$ git new post "blog-name"

#写完了将本地博客推送到github
$ git add .
$ git commit -m "info"
$ git push

# 部署博客
$ git d -g
```

### 参考文档：

​	[利用Hexo在多台电脑上提交和更新github pages博客](https://www.jianshu.com/p/0b1fccce74e0)

