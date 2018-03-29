---
title: hexo-yilia-主题搭建
date: 2018-03-29 20:45:36
tags:
	- hexo
---

- [x] 使用hexo-theme-yilia主题<br>

- [x] 配置头像、打赏码<br>

- [x] 博客文章添加图片<br>
  ![image](/leaf.jpg)

  <!--more-->
## 0x01、添加yilia主题
从github上签出yilia，将theme下的yilia复制到hexo/theme下：
```
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

## 0x02、配置yilia主题
> 配置站点配置文件（hexo根目录下的_config.yml）
```
# 将主题改为yilia
theme: yilia
```
> 配置主题配置文件（theme/yilia/_config.yml）
> [参考Litten博客备份
> ](https://github.com/litten/BlogBackup)


## 0x03、站点图片配置
> 在站点配置文件中配置头像、打赏码等

```
# 将图片存放在hexo/theme/yilia/source下
# 配置hexo/_config.yml
avatar: /icon.jpeg
```

## 0x04、博客图片配置
配置站点配置文件_config.yml:
```
post_asset_folder: true
```
安装上传本地图片插件：
```
npm install hexo-asset-image --save
```
新建博客：
```
hexo n "xxx"
# 在/source/_posts路径下会生成一个xxx.md和xxx文件
```
在md文件中引入图片，将你的图片放入xxx文件夹中：
```
![你想输入的替代文字](xxxx/图片名.jpg)
```



## 0x04、blog截断

在主题配置文件中设置：

```
# yilia/_config.yml文件
excerpt_link: more

# 在博客md文件中，设置截断位置
<!--more-->
```




### 参考文档：
    [hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)