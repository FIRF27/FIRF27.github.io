---
title: Swift混编库的那些事儿
date: 2025-03-07 13:42:21
tags:
---

# Swift混编库的那些事儿

##### 1、在swift framwork中调用oc且不向外暴露

- 使用module.modulemap文件
- `Build Setting` ->`Swift Compile - Search Path` 中设置`module.modulemap`路径 `"$(SRCROOT)/xxx"`

##### 2、提供的oc、swift混编私有.framework是动态库

- 设置 `Build Setting` ->`Build Libraries for Distribution`为`YES` 实现不同版本编译器之间的兼容问题

##### 3、提供的oc、swift混编私有.framework是静态库 且podspec中依赖`Alamofire`这样的第三方库

- 设置 `Build Setting` ->`Build Libraries for Distribution`为`YES` 实现不同版本编译器之间的兼容问题

- 集成方需要在podfile 中加入 以下代码

  ```
  post_install do |installer|
    installer.pods_project.targets.each do |target|
     target.build_configurations.each do |config|
      config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
     end
    end
  end
  ```

##### 4、经过1的操作发现oc工程可以集成成功，swift工程中集成报了2个错 1.`Failed to build module 'SwiftOCDynamicSdk'; this SDK is not supported by the compiler (the SDK is built with 'Apple Swift version 5.5 (swiftlang-1300.0.31.1 clang-1300.0.29.1)', while this compiler is 'Apple Swift version 5.7 (swiftlang-5.7.0.127.4 clang-1400.0.29.50)'). Please select a toolchain which matches the SDK.` 2.No such module ‘OCXXXModule’

**骚操作来了**

- 在swift工程中创建一个

  ```
  module.modulemap
  ```

  文件，并申明一个报错缺失的module

  示例：

  ```
  #这里
  module OCXXXModule 
  {
      export *
  }
  ```

- `Build Setting` ->`Swift Compile - Search Path` 中设置`module.modulemap`路径 `"$(SRCROOT)/xxx"`

#### 5、拓展: Swift组件内调用OC，调用 OC 的私有头文件

5.1 该swift组件需要以二进制方式被另一Swift2混编组件集成

- 原理同4
  5.2 该swift组件需要以以源码方式 被另一Swift2混编组件集成
- import该私有module
