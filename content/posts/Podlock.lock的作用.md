---

title: "Podlock.lock的作用"
date: 2024-10-17T00:25:33+08:00
draft: true

tags: ['ios','cocoapods']

---

最近遇到一个问题，需要跟其他小伙伴对齐一下环境，我就运行了`pod install`和`pod update`。结果发现，前者什么都没做，而后者却安装了一个依赖的新版本。

结论是：

`pod install` 会检查 Podfile.lock文件，这个文件会列出依赖对应的具体版本和CHECKSUMS，也就是说不仅仅是版本，相当于指定了对应的commit了。

`pod update`则是会忽略Podfile.lock指定的内容，直接去更新依赖的最新版本，同时会修改Podfile.lock。

也就是说Podfile.lock和Podfile一样很重要，是必须要提交和保持一致的文件。