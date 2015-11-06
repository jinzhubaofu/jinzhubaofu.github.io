title: code-push 简介
date: 2015-11-05 19:15:23
tags: react-native hot-deploy code-push
---

{% asset_img code-push.png 来自微软的 react-native 热更新服务 %}

大家都知道 native app 的发布周期不是一般的长。从开发到用户更新 app，一般差不多要一个月。正经的互联网 web 应用大概已经升级 4 波了。

所谓热更新，也就是不通过 app store 即更新应用内容。 但是，传统的 native app 是很难做到的。react native 从机制上提供了这种可能。这也是最吸引我的地方。当初还担心苹果会因为这个一刀切死 react-native，从目前看不用担心了。据说苹果官方非常支持 facebook 这么干，还专门提供了一些接口。

之前在美团的 react 分享会上听到他们已经实现了热更新功能。很可惜，他们并没有计划把他们实现开源出来。可能是因为主导的同学都是搞 objective-c 的吧。。。

最近，microsoft 公司推出了一个服务 code push，也就是一个 react-native 的热更新服务。今天，我花一些时间看了一下，整个 codepush 大概分成了三个部分：

## 1. code push cli

这一部分提供了账号相关的服务。`账号`解决了一个非常关键的问题：A app 只能下载到 A app 的 jsbundle，而不能下载到 B app 的 jsbundle。

因此，在 app 运行时，也需要通过账号管理里完成确定这个 app 是哪个账号发布的，这个 app 现在要应该用哪个 jsbundle。

所以，`code push`提供了一个`code push cli`工具，可以这样安装它

```sh
npm install -g code-push-cli
```

然后大家就可以欢快地通过命令行来注册账号、登录、发布应用、发布 jsbundle 等等。

所以呢，code push cli 命令行工具是整个工具的基础。也是发布新版本 jsbundle 的关键哟！

简单地看了一下，具体的实现应该是通过使用诸如```code-push app add MyApp```等命令，在本地项目中生成一些`deploymentkey`和`accesskey`之类的东西，用来在实际产品中免密码进行认证。


## 2. react-native-code-push

这部分是 react-native 应用直接使用的 npm 包。react-native 应用可以直接通过 require 模块的方式，得到 codepush 提供的完整的 api，包括版本检查、下载代码、更新代码等等。

这部分从实现上讲分成了两层：

+ 首先是一个由 javascript 实现的 api 层。向上提供 react-native 应用提供更新接口，向下调用 native 模块实现代码下载和更新。特别的，这一层还需要调用 code-push 的 sdk，完成账号相关的处理。
+ 其次是一个 objective-c 实现的模块，独立于 react-native 应用的 jsbundle。这个部分提供了一个 xcode 的 project，需要通过 xcode 添加。因此，有这一部分的功能存在，codepush 才可以热更新 jsbundle。

## 3. code push service

由于我们需要在用户的手机上更新代码，所以一定需要一个能提供代码(jsbundle)的在线服务。除此之外，还需要提供账户注册、登录（认证/授权）、版本检查等等在线功能。

通过下边这段来自 CodePushConfig.m 的代码可以发现，code push 的在线服务是架设在微软自家的云服务 azure 上的。不知道国内使用起来会不会比较慢。。。

{% asset_img CodePushConfig.m.png CodePushConfig.m %}

好了，简介就到这里了，大家感兴趣的话可以移步 [code push 官方网站](http://microsoft.github.io/code-push/index.html)来获取更多信息。

------------------

PS: 刚刚确认了一下，苹果最近更新了 app 上架条例。之前的条例不允许 app 下载除运行在 wekbit 内部之外的任何可执行代码。但是，最新版本的条例又加上了一个例外：JavascriptCore。JavascriptCore 也就是内建的 js 引擎，也就是 ReactNative 的 [js 执行环境](http://facebook.github.io/react-native/docs/javascript-environment.html#content)。 看上去这个更新就是给 ReactNative 量身定做的啊。下边这段选自苹果的 [app 程序条例](https://developer.apple.com/programs/ios/information/iOS_Program_Information_4_3_15.pdf)：

> 3.3.2 An Application may not download or install executable code. Interpreted
code may only be used in an Application if all scripts, code and interpreters are
packaged in the Application and not downloaded. The only exception to the
foregoing is scripts and code downloaded and run by Apple's built-in WebKit
framework or `JavascriptCore`, provided that such scripts and code do not change
the primary purpose of the Application by providing features or functionality that are
inconsistent with the intended and advertised purpose of the Application as
submitted to the App Store.
