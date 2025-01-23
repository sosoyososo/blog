---
title: "Hugo端口和路径配置"
date: 2023-07-10T18:01:46+08:00
---

其实是第二次折腾hugo了，但还是踩了一些坑，浪费了挺多时间。

### 版本问题
linux下特别是ubuntu下，apt安装的hugo版本比较旧，记得使用github的release里面，安装最新的版本。

<!--more--> 

### 子页面路径上带有端口号的问题
默认情况下，对子页面的引用会是 ```{baseURL:port/pagePath}```的形式，如果hugo服务是通过nigix反向代理到80端口的，我们通常不想让port出现在这里，这时候需要使用 ```--appendPort=false``` 这个参数来避免这个问题。 

没有特别理解这个参数的默认值为啥是true，或者说为啥不是直接删除这个参数，完全信赖baseURL。
