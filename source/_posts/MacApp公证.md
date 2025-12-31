---
title: Mac App公证
tags: [osx, dmg]
categories:
  - [macOS]
date: 2025-12-29 09:34:43
---

# Electron Mac dmg安装包公证流程

[DISTRIBUTE OUTSIDE THE MAC APP STORE](https://help.apple.com/xcode/mac/current/#/dev033e997ca)

[为App签名以通过门禁验证](https://developer.apple.com/cn/developer-id/)

Beginning in macOS 10.14.5, software signed with a new Developer ID certificate and all new or updated kernel extensions must be notarized to run. Beginning in macOS 10.15, all software built after June 1, 2019, and distributed with Developer ID must be notarized. 

Mac App 通过非App Store方式分发, 在10.15会弹窗无法打开移到废纸篓、恶意软件等。

可通过以下命令解决

```
# 禁用Gatekeeper, 允许运行任何来源的应用
$ sudo spctl --master-disable

# 递归移除属性隔离
$ xattr -rc /path/to/xxx.app
```

如果要更优雅的方式， 那么app就必须要进行公证.

!!! 公证的前提.app必须开启Hardened Runtime

###  Electron package.json 设置如下

```
"mac": {
  "icon": "/path/to/target_icon.icns",
  "category": "public.app-category.utilities",
  "hardenedRuntime": true,
  "gatekeeperAssess": false,
  "target": [
   {
    "target": "dmg",
    "arch": ["universal"]
   }
  ],
  "extendInfo": {
   "CFBundleName": "Your app bundle Name",
   "CFBundleDisplayName": "Your App Name"
  }
 }
```



> 如果前3步 都是正常的。 直接跳至4.


## 1.获取本地钥匙串可用证书

```
$ security find-identity | grep 'Developer ID'
eg. 
 2) 35C767AE9F489D19E1D2A22EA25C41F083xxxx "Developer ID Application: xxxx Technology Co., Ltd. (Z369xxxx)"
 其中35C767AE9F489D19E1D2A22EA25C41F083xxxx 为证书指纹
```

## 2. 使用Developer ID证书为app签名

```

$ codesign -fs 35C767AE9F489D19E1D2A22EA25C41F083xxxx -o runtime -v /path/to/targer.app --deep

```

## 3. 生成DMG安装包

```
# brew install  create-dmg
$ create-dmg /path/to/target.app  --no-version-in-filename 
```

## 4. 提交DMG公证

参考[定制公证工作流程](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow)

```
# (本地钥匙串已配置--keychain-profile "notarytool-password")
$ xcrun notarytool submit /path/to/target.dmg --keychain-profile "notarytool-password" --wait

如果成功，则命令行输出如下内容
Submission ID received
  id: 11523ca5-xxx-xxxxx-ac5a-xxxxxxxxx
Upload progress: 100.00% (198 MB of 198 MB)   
Successfully uploaded file
  id: 11523ca5-xxx-xxxxx-ac5a-xxxxxxxxx
  path: /path/to/target.dmg
Waiting for processing to complete.
Current status: Accepted..................
Processing complete
  id: 11523ca5-xxx-xxxxx-ac5a-xxxxxxxxx
  status: Accepted
  
失败， status会是rejected
```

## 5. 验证

通常情况下/dist/mac-universal/xx.app 状态也会变成Accepted

无须验证刚刚提交的dmg文件, 无意义.

```
$ spctl -a -vvv /path/to/target.app
```

## 6. 公证凭据嵌入dmg

```
$ xcrun stapler staple /path/to/target.dmg
$ xcrun stapler validate target.dmp
```
