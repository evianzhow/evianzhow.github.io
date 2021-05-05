---
layout: post
title: DNS Anti-spoofing using NSURLProtocol and HappyDNS
date: '2016-02-19 16:00:54'
tags:
- secpolicycreatessl
- https
- dns
- ios
- happydns
- nsurlprotocol
---

> 2/26/2016 更新1: NSURLConnection 在 iOS 9 中已经被标记为 deprecated，如果你想在生产环境里使用，可以将 NSURLConnection 实现替换为 NSURLSession。具体的代码可以参考 [CustomHTTPProtocol](https://developer.apple.com/library/ios/samplecode/CustomHTTPProtocol/Listings/Read_Me_About_CustomHTTPProtocol_txt.html)。

> 2/26/2016 更新2: NSURLProtocol 可能会导致某些情况下的意外崩溃，这与 NSURLProtocol 本身实现没有关系，可能与系统内部有一定的冲突，具体原因还在调查中。

最近在参与一个 iOS 项目的时候，遇到了一点部署在 AWS 上服务器 IP 地址解析上的问题，表现是大量用户反馈出现了 -1003 错误，即 *NSURLErrorCannotFindHost*，而且这个错误在蜂窝数据下和 WiFi 环境下都有发生。

服务器的 DNS 解析是由 Amazon 的 Route 53 来做的，搜索了相关的关键词，也发现了遇到类似的 [问题](https://github.com/aws/aws-sdk-ios/issues/167) 的开发者。暂时保留对 Route 53 在国内可用性的怀疑，从客户端方面着手解决。

想解决这个问题，方法很简单，让原来通过域名访问服务器的方式，改成使用 IP 地址访问。

在项目中使用七牛的 [Objective-C SDK](https://github.com/qiniu/objc-sdk) 时，看到其依赖了一个有趣的库叫 [HappyDNS](https://github.com/qiniu/happy-dns-objc)。其原理为一个 DNS Resolver，想必七牛的 SDK 也遭遇到与我类似的问题才诞生这么一个库。

那么，我们可以在程序中内置一个 DNS Resolver，剩下的事情就是让 App 使用 IP 地址访问我们的服务器了。在 HappyDNS 的 Issues 列表里，我们发现了一些非常有用的 [讨论](https://github.com/qiniu/happy-dns-objc/issues/1)，关于利用 HappyDNS 来解决 DNS 解析不正确 / 劫持的方法，作者推荐使用 NSURLProtocol 来完成 NSURLRequest 的域名到 IP 的转换过程。

如果你对 NSURLProtocol 还不熟悉（*这当然很正常！这是一个非常冷门的 Abstract Class！*），可以通过 Ray Wenderlich 的 [NSURLProtocol Tutorial](http://www.raywenderlich.com/59982/nsurlprotocol-tutorial) 教程对这个类能干什么和它的基本结构有一个大致的了解。

在教程的 Demo 里，作者给我们演示了如何截获和更改我们的网络请求对象。阅读到这里，满怀欣喜的尝试了一下，果然看上去没有问题。原以为解决了问题，但是，如果是那么简单，也没有必要出现这篇文章了。

如果你的服务器仅仅通过 HTTP 协议访问，那么接下来的内容可以直接略过了，遵循教程里的方法，外加使用

```objc
[newRequest setValue:originRequest.URL.host forHTTPHeaderField:@"Host"];
```
将 HTTP 头部的 Host 字段设置为原来的地址，即可完美解决问题。

但是，

**如果你的服务器 API 是通过 HTTPS 访问的话，那么更改 URL 会直接导致 SSL 验证不通过。**

我们都知道，在 [HTTPS 请求过程](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html) 中，会有一个协商 Session 密钥的过程。

<img src="http://image.beekka.com/blog/2014/bg2014092002.png" />
图片来自 [图解SSL/TLS协议](http://image.beekka.com/blog/2014/bg2014092002.png)

在这个过程中，假设我们访问的 URL 是 https://www.google.com/api ，同时 Google 服务器发送的证书是 *.google.com 的这么一个 Hostname 的话，验证通过。但我们在 NSURLProtocol 子类里更改了请求地址，变成了一个这样的地址 https://216.58.221.36/api ，但是这时访问 Google 服务器的 443 端口时，服务器仍返回的是 *.google.com。此时验证就会不通过，你可以利用浏览器测试一下，就会发现提示证书的 Host name mismatch。

<img src="https://static.evianzhow.com/google_ssl_mismatch.png" />

对于这样的问题，Apple 官方在 [Technical Note TN2232](https://developer.apple.com/library/ios/technotes/tn2232/\_index.html#//apple_ref/doc/uid/DTS40012884-CH1-SECSERVERNAME) 里并不推荐你采用忽略错误（Disabling server trust evaluation）这一做法，这样会极大的降低 HTTPS 的安全性。其推荐在 NSURLProtocol 的 delegate 的函数中使用 Security framework 的函数添加正确的 SSL 地址名来解决。

参考了许多资料之后，总结出大致的思路就是：

  1. 重写 NSURLProtocol 子类中 NSURLConnectionDelegate 的 `- (void)connection:(NSURLConnection *)connection willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
` 方法
  2. 获取上下文中的 Server Trust 对象，对其有效性进行验证，如果合法则创建对应的 Crediental Object 并继续连接
  3. 如果 Server Trust 不合法，但是是处于可以恢复的状态（kSecTrustResultRecoverableTrustFailure），在调用中使用 `SecPolicyCreateSSL` 创建一个新的 Policy 然后应用到 Trust 上，重新尝试步骤 2

思路明确之后，解决问题就变得比较容易了，但是还是实现的过程中出现了一些小小的问题，这里分享给大家。

1. 调用 SecPolicyCreateSSL 函数后还是无法通过验证

  根据 Stack Overflow 上 [Overriding TLS server validation hostname doesn't seem to be working](http://stackoverflow.com/questions/23786259/overriding-tls-server-validation-hostname-doesnt-seem-to-be-working) 这个回答，这里函数的 hostname 应该是客户端用于验证的叶证书名称，在用 IP 地址替换域名的请求里，这里原来的 hostname 应该是 IP 地址，使用被替换的域名作为参数即可。

2. kCFStreamErrorDomainSSL, -9802 错误

  这个错误并不是因为 NSURLProtocol 的实现错误造成的，而是由于 iOS 的 App Transport Security 不支持你的服务器上的加密方法造成的，解决方法为：添加加密方法或者在 Info.plist 里启用 `NSAllowsArbitraryLoads`

相关代码如下:

```objc
- (void)connection:(NSURLConnection *)connection willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
{
    SecTrustRef trust = challenge.protectionSpace.serverTrust;
    SecTrustResultType result;
    NSURLCredential *cred;

    OSStatus status = SecTrustEvaluate(trust, &result);

    NSInteger retryCount = 1;
    NSInteger retainCount = 0;

    while (status == errSecSuccess && retryCount >= 0) {
        if (result == kSecTrustResultProceed || result == kSecTrustResultUnspecified) {
            cred = [NSURLCredential credentialForTrust:trust];
            retainCount > 0 ? CFRelease(trust) : nil;
            [challenge.sender useCredential:cred forAuthenticationChallenge:challenge];
            return;
        } else if (result == kSecTrustResultRecoverableTrustFailure) {
            retryCount--;

            CFIndex numCerts = SecTrustGetCertificateCount(trust);
            NSMutableArray *certs = [NSMutableArray arrayWithCapacity:numCerts];
            for (CFIndex idx = 0; idx < numCerts; ++idx) {
                SecCertificateRef cert = SecTrustGetCertificateAtIndex(trust, idx);
                [certs addObject:CFBridgingRelease(cert)];
            }

            SecPolicyRef policy = SecPolicyCreateSSL(true, (__bridge CFStringRef)originalHostname);
            OSStatus err = SecTrustCreateWithCertificates((__bridge CFTypeRef _Nonnull)(certs), policy, &trust);
            retainCount++;
            CFRelease(policy);

            [certs removeAllObjects];
            certs = nil;

            if (err != noErr) {
                NSLog(@"Error creating trust: %d", err);
                break;
            }
            status = SecTrustEvaluate(trust, &result);
        }
    }

    [challenge.sender cancelAuthenticationChallenge:challenge];
}
```
完整的 Demo Project 在 [GitHub](https://github.com/Her0n/DNS-Anti-Spoofing-Example) 上以供大家参考，同时包括了 HappyDNS 的使用。

如果你还有不清楚的地方，欢迎留言与我交流。

#### References

- http://oncenote.com/2014/10/21/Security-1-HTTPS/
- http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html
- https://developer.apple.com/library/ios/technotes/tn2232/\_index.html#//apple_ref/doc/uid/DTS40012884-CH1-SECSERVERNAME
- http://lists.apple.com/archives/cocoa-dev/2013/May/msg00354.html
- https://github.com/AFNetworking/AFNetworking/issues/1157
- http://stackoverflow.com/questions/23786259/overriding-tls-server-validation-hostname-doesnt-seem-to-be-working
- http://stackoverflow.com/questions/30778579/kcfstreamerrordomainssl-9802-when-connecting-to-a-server-by-ip-address-through

如果本文对你有帮助的话，不妨通过支付宝可以请我喝一杯咖啡！ :)
<img src="https://static.evianzhow.com/alipay.jpg" alt="austinchou0126@gmail.com" />