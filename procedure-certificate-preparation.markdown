# Procedure: Certificate Preparation

This page contains information about certificate preparation procedure.


## 1. 证书知识及准备工作

证书是由 Apple 官方颁发，用以证明开发者身份的特殊文件。在 iOS 开发中主要用于代码签名，保障 iOS 生态的健康安全，分为开发者证书和发布者证书。只有在本机模拟器调试时无需代码签名，当 App 需要在真机运行和发布时需要使用相应证书进行签名。

要获得证书，首先需要拥有相应权限的开发者帐号，通过在本地生成配对的密钥。然后向 Provisioning Portal 提交公钥后换取，后续证书在使用时会验证本地私钥

在 Xcode 中，使用描述文件（Provision Profile）是一个包含调试者证书、授权设备清单、应用 ID 等信息的文件。使用描述文件，在 Build Settings 中选择存于 Keychain Access 中的证书文件，设置调试和发布任务时的代码签名，即可使用证书对代码进行签名。

如果要将生成的私钥共享给团队成员，可以在 Keychain Access 中找到导入的证书，右击导出为包含私钥的 Personal Information Exchange (.p12) 文件即可，文件导出时可以创建密码。团队成员再导入 .p12 证书后就完整包含了证书和私钥。


## 2. 各流程中证书的需求情况

以下是各流程中证书的需求情况：

* __模拟器调试__：不需要证书
* __真机调试__：描述文件（Provisioning Profiles），开发者证书（`ios_development.cer`）
* __打包和发布__：描述文件（Provisioning Profiles），可用于发布的开发者证书（`ios_distribution.cer`）
* __消息推送后端服务__：apns 证书


## 3. 开发中常见的证书及文件

Below are common seen certificates and related files.


### 3.1 CSR 文件

CSR (Certificate Request) 文件用于换取证书的公钥文件，实际是在本地基于 RSA 加密得到配对的密钥，私钥存于 Keychain Access 用于签名，公钥作为换取证书的凭证。

要生成 CSR 文件，可以使用OSX 系统自带的 Keychain Access 或者在命令行下使用 openssl 生成。

#### 3.1.1 使用 Keychain Access 生成 CSR 文件

首先打开 Keychain Access 应用（按 "Ctrl+Space" 打开 Spotlight 搜索，然后输入 `Keychain Access` 就可以找到了）。

然后在任务栏上找到 "Request a Certificate From a Certificate Authority…" 并选择。如下图所示：

![certificate_01](http://gitlab.djicorp.com/uploads/david.qiu/learning-ios/50d02c2cfd/certificate_01.jpeg)

最后输入 email 等信息后保存为 .certSigningRequest 格式文件即可。

#### 3.1.2 使用 openssl 生成 CSR 文件

在命令行下输入如下命令即可：

```
$ openssl genrsa -out private.key 2048
$ openssl req -new -sha256 -key private.key -out my.certSigningRequest
```


### 3.2 开发者证书

开发者证书由 Apple 官方颁发，是用来证明开发者资格的证书文件，有 "开发证书" (`ios_development.cer`) 和 发布证书 (`ios_distribution.cer`) 两种类型。开发者证书跟用于开发的 Mac 上的私钥绑定，只能在拥有私钥的机器上使用。如果要迁移机器，则需要将证书导出为 .p12 文件。

在 [Apple 开发者中心](http://developer.apple.com/) 的 "Certificates" 面板中，点击 "添加 Certificate" 并上传刚刚生成的 CSR 文件，即可获取 `ios_development.cer` 证书。


### 3.3 apns 证书

apns (Apple Push Notification Service) 证书用于服务端消息推送，类似 ssl 证书使用，和 App 端的开发打包没有关系。

在 [Apple 开发者中心](http://developer.apple.com/) 的 "Identifiers" 面板中，点击 "添加 App ID" 并上传刚刚生成的 CSR 文件，即可获取 `aps_production.cer` 证书。


### 3.4 .p12 证书

.p12 (Personal Information Exchange) 证书实际是包含了 cer 证书及私钥信息，可以分发给团队成员。

在 Keychain Access 中找到已经导入的 cer 证书，然后点右键导出为 .p12 格式文件即可。生成的时候可以设置密码。


### 3.5 描述文件

描述文件 (Provisioning Profiles) 是一个包含 Certificate、AppID、Devices ID 的文件，用于在 Xcode 调试打包时提供授权的配置信息。

在 [Apple 开发者中心](http://developer.apple.com/) 的 "Provisioning Profiles" 面板，点击 "添加 iOS Provisioning Profiles" 并上传刚刚生成的 CSR 文件，即可获取 `.mobileprovision` 文件。

同时，可以在 Xcode 中登录开发者帐号后，可以连接开发者中心直接获取。


## 4. 几种开发者帐号区别

开发者帐号分以下几种：

* 个人（Individual） __$99/year__：个人帐号可以进行真机调试，发布到 AppStore，每年最多为 100 台设备分发。
* 公司（company） __$99/year__：公司帐号和个人帐号类似，只有这两种帐号可以发布到 AppStore，主要特权是可以添加多个开发者子账号，但只允许主账号进行提交、发布等操作，在协同开发时比较灵活，可以各自管理授权设备等。
* 企业（enterprise） __$299/year__：企业帐号无法用于 AppStore 发布，但可以不通过 AppStore 发布任意 iOS 都可以安装的应用。
* 大学（University） __free__：大学帐号不能发布到 AppStore，主要拥有真机调试的权限。


## 5. Resources

Below are related resources about the certificate preparation procedure issues.

* [iOS 开发流程笔记](https://github.com/leecade/ios-dev-flow)
* [Mac 环境下生成 Apple 证书教程](http://open.zhui.cn/index.php?title=MAC环境下生成Apple证书教程)
* [苹果Xcode 证书生成、设置、应用完整图文教程](http://blog.sina.com.cn/s/blog_5a6efa330101dx05.html)


