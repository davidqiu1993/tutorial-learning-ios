# Procedure: Certificate Preparation

This page contains information about certificate preparation procedure.


## 1. 证书知识及准备工作

证书是由 Apple 官方颁发，用以证明开发者身份的特殊文件。在 iOS 开发中主要用于代码签名，保障 iOS 生态的健康安全，分为开发者证书和发布者证书。只有在本机模拟器调试时无需代码签名，当 App 需要在真机运行和发布时需要使用相应证书进行签名。

要获得证书，首先需要拥有相应权限的开发者帐号，通过在本地生成配对的密钥。然后向 Provisioning Portal 提交公钥后换取，后续证书在使用时会验证本地私钥

在 Xcode 中，使用描述文件（Provision Profile）是一个包含调试者证书、授权设备清单、应用 ID 等信息的文件。使用描述文件，在 Build Settings 中选择存于 Keychain Access 中的证书文件，设置调试和发布任务时的代码签名，即可使用证书对代码进行签名。

如果要将生成的私钥共享给团队成员，可以在 Keychain Access 中找到导入的证书，右击导出为包含私钥的 Personal Information Exchange (.p12) 文件即可，文件导出时可以创建密码。团队成员再导入 .p12 证书后就完整包含了证书和私钥。


### 1.1 各流程中证书的需求情况

以下是各流程中证书的需求情况：

* __模拟器调试__：不需要证书
* __真机调试__：描述文件（Provisioning Profiles），开发者证书（`ios_development.cer`）
* __打包和发布__：描述文件（Provisioning Profiles），可用于发布的开发者证书（`ios_distribution.cer`）
* __消息推送后端服务__：apns 证书


### 1.2 开发中常见的证书及文件

#### 1.2.1 CSR (Certificate Request)

CSR (Certificate Request) 文件用于换取证书的公钥文件，实际是在本地基于 RSA 加密得到配对的密钥，私钥存于 Keychain Access 用于签名，公钥作为换取证书的凭证。

要生成 CSR 文件，可以使用以下方法的其中一种：

__方法一：使用 OSX 系统自带的 Keychain Access__

* 打开 Keychain Access 应用（按 "Ctrl+Space" 打开 Spotlight 搜索，然后输入 `Keychain Access` 就可以找到了）

* 选择 "Request a Certificate From a Certificate Authority…"

  ![certificate_01](http://gitlab.djicorp.com/uploads/david.qiu/learning-ios/50d02c2cfd/certificate_01.jpeg)

* 输入 email 等信息后保存为 .certSigningRequest 文件

命令行下使用 openssl 生成

$ openssl genrsa -out private.key 2048
$ openssl req -new -sha256 -key private.key -out my.certSigningRequest





