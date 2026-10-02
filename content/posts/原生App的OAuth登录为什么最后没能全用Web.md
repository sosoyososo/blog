---
title: "原生 App 的 OAuth 登录，为什么最后没能全用 Web"
date: 2026-10-02T18:10:00+08:00
tags:
  - OAuth
  - Mobile
  - Web
---

这个项目本身是一个 Web 业务，后来用 Capacitor 封装成了 App。登录这块 OAuth 已经在 Web 端跑通了，所以最初的设想很顺理成章：App 里不做登录，直接复用 Web 的 OAuth。

流程设想是：App 跳转到 OAuth 授权 → 用户登录 → 授权服务 redirect 回 callback URL → App 接管，处理登录态。

前面几步都没问题，Web 实现完全够用。问题出在**最后一跳**。

<!--more-->

## 第一次尝试：应用内浏览器

用应用内浏览器（SFSafariViewController / Chrome Custom Tab）承载整个 OAuth 流程。登录完成，授权服务 redirect 回 App 自己的 Universal Link / App Link。

结果：回不来。

原因是应用内浏览器和 App 属于同一个包。系统不允许从应用内浏览器程序化地跳回同包的 App，用户就卡在浏览器里了——登录明明成功了，但回不到 App。

## 第二次尝试：跳系统默认浏览器

改用 `window.open(url, '_system')`，把 OAuth 放到系统默认浏览器里做。

这一步能跳出去，但授权服务的回调会落到系统默认浏览器。这时不同浏览器开始分化：

- **Safari**：能正常跳回 App
- **Chrome**：跳不回，停在浏览器里

也就是说，最后一跳能不能成，取决于用户系统默认浏览器是什么。这不是靠 Web 代码能控制的事情。

## 为什么 Web 做不到

最后一跳本质上是"浏览器要不要把当前 URL 交给 App"的决定，这个决定属于平台和浏览器，不属于 Web API。

Safari 对 Universal Link 的 App 切换支持完整，会提示或直接打开对应 App。而 Chrome（尤其在 iOS 上基于 WKWebView 封装）不会稳定地对 OAuth 回调链上的导航弹出"在 App 中打开"。更麻烦的是，Web 层面没有办法去检测、统一或者绕过这个行为。

所以结论是：Web 实现的 OAuth 可以覆盖整个流程，唯独没法保证最后那个回调稳定落到 App 里。而对原生 App 来说，这个保证恰恰是登录存在的意义。

## 最终方案

没办法，OAuth 这一段改成用 native 插件实现。登录流程在原生上下文里跑，回调直接以原生 deep link 的形式回到 App，不依赖浏览器的 App 切换行为。前面几个步骤的 Web 复用也就到此为止了。

## 回头看

如果一开始就知道"最后一跳"是一个平台约束、Web 做不到，就不用在 Web 的 OAuth 上试这么久了。复用 Web 登录的想法本身没问题，问题的边界在登录的最后一步，而最后一步恰恰是最依赖平台行为的一步。
