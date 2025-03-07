---
title: iOS制作私有库
date: 2025-03-07 13:42:02
tags: [iOS]
categories: 
  - [iOS]
---

# iOS制作私有库

大致步骤入下:

- 创建远程索引库、远程私有库
- 创建远程索引库时，最好同时初始化一个readme文件。否者后面操作会报错，解决也是需要在远程索引仓库创建一个空文件
- 创建本地私有库、关联远程私有库
  - 终端输入`pod lib create xxxSDK `
  - `cd xxxSDK`
  - `git remote add origin https:xxx.xxxSDK.git`
- 本地添加私有库索引
  - 终端输入`pod repo add name[索引名] source [远程索引地址] `
  - 例如：`pod repo add PluginPodspecs http://xxx.pluginpodspecs.git`
  - `pod repo list`查看
- 添加本地私有库内容、修改`xxx.podspec`文件
- 本地校验
  - `pod lib lint --verbose --allow-warnings `
  - 如果私有库里有嵌套另一个私有库 在上面命令后面在加上参数`--sources=[索引库址],https://github.com/CocoaPods/Specs.git `
- 私有库提交到远程仓库、打tag
  - `xxx.podspec`文件具体可查看官方文档 https://guides.cocoapods.org/syntax/podspec.html
- 远程校验
  - `pod spec lint --allow-warnings --verbose`
- 提交本地索引到远程
  - `cd xxxSDK && pod repo push [本地索引库] [索引文件名]`
  - `pod repo push PluginPodspecs xxxSDK.podspec --verbose --allow-warnings`
