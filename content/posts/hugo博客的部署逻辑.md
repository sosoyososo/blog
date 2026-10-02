---
title: "hugo博客的部署逻辑"
date: 2026-10-02T17:58:00+08:00
tags:
  - Hugo
  - GitHub
  - Deployment
---

之前写过一篇[《一个比较诡异的部署逻辑》](/posts/一个比较诡异的部署逻辑/)，记录的是一段很折腾的历史：为了绕开备案，域名解析到一台海外 vps，再由 nginx 反向代理到内地另一台机器上的 hugo 服务。绕了一大圈，代价是要自己盯着一台可能 OOM 的机器。

现在这套折腾终于可以收掉了，整个链路简单到有点无聊。

<!--more-->

## 现在的链路

```
本地写 markdown
    ↓ git push
GitHub Actions (master 分支触发)
    ↓ hugo --minify
编译成静态资源，push 到 gh-pages 分支
    ↓
GitHub Pages 提供访问
    ↓
Cloudflare DNS: blog.karsa.info → GitHub Pages
```

整条链路上没有存储资源、没有 api server、也不需要自己管理证书。证书是 GitHub Pages 自动签发的 Let's Encrypt 证书，到期自动续期。

## 几个关键点

**内容仓库就是源仓库。** markdown 直接写在 `content/posts/` 下，push 就完事，不需要在本地先 build 再传。`public/` 已经在 `.gitignore` 里，编译产物不进版本库。

**主题是 submodule。** `themes/smigle` 通过 git submodule 引入，所以 workflow 里 checkout 那一步需要 `submodules: true`，否则构建会直接失败。这是一个很容易踩的坑——本地能跑，CI 上起不来，原因就是主题没拉下来。

**CNAME 放在 `static/`。** hugo 构建时会把 `static/` 下的文件原样拷贝到输出目录，`static/CNAME` 就这样跟着进到了 gh-pages 分支根目录，GitHub Pages 靠它识别自定义域名。CNAME 必须在仓库里（而不是只在 Pages 后台设置），否则每次重新部署都有丢失的风险。

**`baseURL` 必须和实际域名一致。** `hugo.toml` 里写的是 `https://blog.karsa.info/`，它决定了生成页面里所有链接的绝对前缀。改域名的话，这里要同步改，否则页面内的跳转和静态资源路径会全错。

**hugo 需要 extended 版本。** 主题用了 SCSS，标准版编译不了，workflow 里 `extended: true` 不能去掉。

## 域名解析

Cloudflare 那边只需要给 `blog.karsa.info` 加一条 CNAME 记录，指向 `sosoyososo.github.io`（`<user>.github.io`）。不需要开代理（灰云），让流量直接走 GitHub Pages 就行——这样 GitHub 才能签发证书，开了代理反而会拿不到。

DNS 生效之后，GitHub Pages 后台的 Custom domain 里填 `blog.karsa.info`，等它验证通过、自动签发证书即可。之后证书续期、HTTP/2、CDN 都不用操心。

## 小结

之前那套方案的核心动机是"不备案"，而 GitHub Pages 本身就架在境外，天然没有这个问题。既然不需要在境内落地机器，也就没有了备案的约束，那么直接在境外托管静态站点就是最优解。绕了一大圈之后，答案其实一直很简单：**能静态化的就静态化，然后交给平台**。
