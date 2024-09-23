---

title: "XCode调试iOS时候证书配置的一个坑"
date: 2024-09-24T02:09:32+08:00

---


调试iOS应用的时候，编译成功，安装app到设备也成功，启动app也成功，但Xcode自动结束调试，报错如下：

```
Could not attach to pid : “20860”

attach failed (Not allowed to attach to process.  Look in the console messages (Console.app), near the debugserver entries, when the attach failed.  The subsystem that denied the attach permission will likely have logged an informative message about why it was denied.)
```


![报错截图](https://res.karsa.info/files/file/server/pay-record-icon/2024/September/24/1727115080501333189)

先是网络上搜索了一圈，没有发现什么有用线索，之后有一瞬想到证书配置。我配置这个app的证书时候配置错了，在debug模式下使用的distrution的证书和对应的provisioning profile。这个问题我是知道的，但之前一段时间一直都是RN和Capacitor代码，跑起来打包上就抛弃xcode了，也没发现有什么问题，好了现在知道了。

![错误原因](https://res.karsa.info/files/file/server/pay-record-icon/2024/September/24/1727115873060040526)