# Procedure: Online Debugging

真机调试指 Mac 连上 iPhone 后，Xcode 可以直接以这台 iPhone 设备为 Build Target，能在 iPhone 里执行编译结果。

真机测试可分为拥有独立开发者帐号（也包括公司帐号或企业帐号成员）和共享开发者帐号两种情况。


## 1. 拥有独立开发者帐号

流程如下：

1. 在 Provisioning Portal 新建应用，配置授权设备等
2. 开发机上导入证书
3. 在 Xcode 上登录开发者帐号，不需要准备描述文件，Xcode 会自动生成（如果是公司帐号可以自动生成 iOS Team Provisioning Profile）


## 2. 共享开发者帐号

如果无法在 Xcode 登录一个开发者帐号，也可以通过他人对你手机和 App ID 进行授权。得到 `.mobileprovision` 描述文件再导入其含私钥的证书（.p12 证书）即可。具体步骤如下：

1. 获得手机的 UDID （可以连上电脑，在 iTunes 中查看）
2. 告知对方 UDID （用以设备授权） 和 App ID
3. 得到对方生成的证书和描述文件后，先导入 .p12 证书，再双击 .mobileprovision 描述文件进行打开
4. 连接手机，在 Xcode 中选择 Build Target 为已连接的手机


## 3. Resources

Below are related resources about the online debugging distribution procedure.

* [iOS 开发流程笔记](https://github.com/leecade/ios-dev-flow)


