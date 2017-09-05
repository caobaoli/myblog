---
title: NexT主题个性化设置
date: 2017-08-23 17:19:06
tags: 学术
categories: Hexo主题
---

#### 站点配置文件与主题配置文件：
1. 站点配置文件是hexo根目录下的`_config.yml`
2. 主题配置文件是主题（theme）文件夹下的`_config.yml`

#### 添加分类、标签：
1. 在站点跟目录下，打开git bash，输入
	
		hexo new page "categories"

	之后在站点目录中的source文件夹，会新增一个categories的文件夹里面有一个index.md，内容如下：

		title: categories
		date: 2017-08-07 23:57:50
		type: "categories"

	我们可以给index.md中添加(这样当我们给博客添加评论功能时，此页面就不包含)：

		comments: false

2. 创建标签页面类似
	
		hexo new page "tags"

#### NextT主题重点设置(主题配置文件_config.yml)
1. 设置图像栏

		avatar: your avatar url
	your avatar url可以是完整的互联网URL，也可是本地图片路径即你在next\source\uploads\avatar.png
2. 设置favicon图标
		
		favicon: /uploads/favicon.ico
3. 首页文章以摘要形式显示

		auto_excerpt:
  			enable: true
  			length: 150
	注意`length`代表显示摘要的截取字符长度
4. 社交关联

		social: github: https://github.com/caobaoli

#### [具体参考NtxtT官方文档配置](http://theme-next.iissnan.com/theme-settings.html)