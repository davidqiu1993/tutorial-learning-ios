# Procedure: Alpha Test Distribution

This page contains information about alpha test distribution procedure.


## 1. App Distribution

当 App 开发进行到一定程度, 需要更多的人参与测试, 需要谋求一种方式方便应用能安装进更多的设备中

进行内测发布主要的关键点，是如何将应用打包为 .ipa 安装包。Xcode 6 以后，个人或公司帐号无法对应用打包为 .ipa 安装包，要么用 Xcode5 打包，要么拥有企业帐号级别的授权。个人和公司帐号权限只有在 TestFlight 或者越狱渠道下才能完成不授权安装；企业帐号授权可以在 ad-hoc 或者 in-house 渠道下分发，完成不授权设备安装。


### 1.1 ad-hoc

打包时必须在 Xcode 登录企业帐号（或其成员帐号），在并已导入证书和描述文件的情况下进行。任何用户（未授权）都可以在手机上用浏览器访问一个 url 完成安装，例如：`itms-services://?action=download-manifest&url=https://example.com/manifest.plist`。

ad-hoc 的最大的问题是安装量有 100 的上限，无法作为一个量很大的分发渠道。


### 1.2 in-house

针对企业内部用户进行分发，相比 ad-hoc 无安装量上限。

> iOS 8.1.3 开始不能用企业证书 iResign 方式重新签名的应用无法安装（https://support.apple.com/en-us/HT204245）。


### 1.3 TestFlight

仅支持 iOS 8.0 以上，不需要对设备 UDID 进行授权，适合个人和公司开发者。在应用发布前可以开启 TestFlight Beta 测试，并添加测试者的 iTunes Connect 帐号。需要待测用户拥有 iTunes Connect 帐号并在设备安装 TestFlight 客户端才能使用。

这种方式非常便于推送应用更新和收集测试信息。


### 1.4 越狱安装

如果测试设备都越狱了，这种方式非常灵活简单，只有能导出 .ipa 安装包就能通过 itools 等第三方工具安装了。


## 2. Resources

Below are related resources about the alpha testing distribution procedure.

* [iOS 开发流程笔记](https://github.com/leecade/ios-dev-flow)


