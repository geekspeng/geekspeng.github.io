---
title: hexo 下的分类和表签无法显示
date: 2017-09-01
updated: 2017-09-01
tags: [hexo]
categories: [hexo]
---

#### 在命令行中输入hexo new page tags

```
$ hexo new page tags
```
#### 这时会在在sources/tags里面有个index.md的文件，打开这个文件编辑

```
---
title: tags
date: 2017-08-28 08:33:46
type: "tags"
---
```
type: 改成tags

<!-- more --> 

#### 在主题配置文件中，在menu项下，要把tags页打开如

```
menu:
  home: /
  archives: /archives
  tags: /tags    //确保标签页已打开  
  categories: /categories  
  about: /about
```

#### 在你要发布的文章中添加标签

```
---
title: hexo 下的分类和表签无法显示
date: 2017-08-28 08:33:46
tags: [hexo]
categories: [hexo]
---
```

所有冒号后面都有个空格




