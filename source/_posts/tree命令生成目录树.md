title: tree 命令生成目录树
date: 2018-02-28 09:18:07
tags:
- 笔记
---

有时，我们需要将项目的结构以树的形式罗列，如下图。在window下，可以使用tree命令。linux系统的可以看文章后面的扩展阅读了解。


tree 命令默认**只显示子目录名**，并不会展示目录里面的文件名，加上参数 **/F** 才能显示完整的目录树。
如果需要输出至文件则使用如下命令(输出至test.txt文件)

> tree /F > test.txt

<!--more-->
查看tree命令的其他参数：
> tree /?

参数：
- /F 显示文件夹中每个文件的名字
- /A 使用ASCII字符，而不使用扩展字符。


```
│  app.js
│  app.json
│  app.wxss
│  project.config.json
│  test.txt
│  
├─pages
│  ├─index
│  │      index.js
│  │      index.wxml
│  │      index.wxss
│  │      
│  └─logs
│          logs.js
│          logs.json
│          logs.wxml
│          logs.wxss
│          
└─utils
        util.js
```

## 扩展阅读
[Linux下的tree命令 --Linux下目录树查看](http://blog.csdn.net/zjf280441589/article/details/39960147)