---
title: "SSL Pin 研究和原理以及在iOS调试的坑"
date: 2024-03-29T13:12:17+08:00
draft: true
---

最近碰到个在 ReactNative 里面使用 SSL pin 的坑，困扰了我许久，解决后记录下。

## 什么是 SSL，安全性怎么保障

https 我们很熟悉，iOS 是默认强制使用的，https 本质是运行在 ssl 加密通道上的 http，核心是 ssl 加密。ssl 加密是通过服务端下发共用证书，协商好加密方案后建立的加密通道。
安全性是通过系统内置的知名 CA 的根证书，与建立 SSL 通道时候，服务端下发的证书链进行比对，来保障。这里默认 CA 是可以信任的，那么 CA 签发的证书就是可信的。CA 下发的证书可以跟 CA 的根证书形成证书链条，用以进行证书有效性确认。安全性得以保障。

## SSL pin 是什么，用在什么场景

ssl 加密通道是非对称加密通道，服务端下发了公钥，客户端进行加密，服务端拿私钥进行解密，本质上是信任了 CA 签发的证书，也就是服务端作为主导。
但 CA 有时候也不靠谱，万一证书签发错误，或者泄漏给别人了，咋办？SSL pin 这时候就是用来进行额外的保障。
SSL pin 通过在客户端内置公钥证书，和服务端下发的证书做比对，在系统比对的基础上，增加一层自己比对的过程。自己比对的时候，就可以在 CA 证书之外，有自签名证书作为额外一层保障。这时候，即使对方拿到了 CA 签发的正确证书，我们自己签发的证书也可以保障服务的安全性。

## iOS 中 SSL pin 怎么用

iOS 中用 NSURLSession 的时候，指定一个实现了`URLSession:didReceiveChallenge:completionHandler`方法的 delegate，就可以在回调方法里，进行自定义的证书比对过程。
整个过程相对比较麻烦，所以现在大家用的比较多的是通过一个开源库`TrustKit`进行比对。
TrustKit 通过配置域名和与之对应的公钥证书签名，在 SSL 通道建立的时候，获取到证书链，比对证书链中每个证书，一旦有比对成功的，就算是成功，如果所有的都比对失败，就算是 SSL pin 失败了。SSL pin 失败，会打断 ssl 通道的建立，所以后续 ssl 通道上的 http 请求也不会发生。

## React Native 怎么用 ssl pin

React Native 的网络请求是通过 React-RCTNetworking 这个库来保证的，里面有三个相关具体的请求，`RCTHTTPRequestHandler`是 js 网络请求的本地实现。他就是通过 NSURLSeesion 来进行网络请求的，并且把代理设置成了自己。但他没有实现`URLSession:didReceiveChallenge:completionHandler`这个方法，所以默认就没有对 ssl pin 进行支持。
对它进行支持，我们可以实现 RCTHTTPRequestHandler 的 category，增加这个方法，记得保证这个 category 在 js 网络请求之前被加载，最简单的是实现一个空方法，在初始化的时候调用一下。

## 调试遇到的坑

调试的时候上述方法都用了，但 SSL pin 就是不起作用，新增的方法都不被调用，怀疑了 xcode 的 bug，怀疑了 iOS sdk 内部实现不一致，怀疑了 RN 是否屏蔽了他。验证这些都没问题，花了我 99%的时间。
最终发现 NSURLSeesion 生成时候使用的 NSURLSessionConfiguration，默认的生成方法 defaultSessionConfiguration 会使用一些缓存规则，导致创建 session 的时候，有可能复用以前的 ssl 通道，导致 ssl pin 失效。
所以调试时候，可以使用 ephemeralSessionConfiguration 来生成 NSURLSessionConfiguration。
后来发现这也是 TrustKit 自己 demo 里面生成 session 的配置。

## 后记

原因找到后很简单，但过程值得回味。怀疑权威是对的，但在怀疑权威之前，最后先验证自己是否正确，顺序搞错，就是这次精力消耗掉事故的本质原因。
