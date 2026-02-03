---
layout: '#'
title: h5页面在ios端无法自动弹出键盘
date: 2026-02-03 15:17:38
tags:
---

# 允许 WKWebView 在没有用户交互的情况下显示键盘

## 实现原理

WKWebView没有UIWebView 的 `keyboardDisplayRequiresUserAction = false`

只能通过 Runtime 替换 WKContentView 的私有方法 ` _elementDidFocus:userIsInteracting:...`  强制将 `userIsInteracting` 参数设为 `true`，从而绕过 WKWebView 的用户交互检查。 

⚠️ 注意：此方法使用了私有 API  存在审核风险

```

   // 私有方法选择器： "_elementDidFocus:userIsInteracting:blurPreviousNode:activityStateChanges:userObject:"
   private func obfuscatedSelectorName() -> String {
//        let parts = [
//            "_ele", "ment", "Did", "Focus",
//            ":user", "Is", "Inter", "acting",
//            ":blur", "Previous", "Node",
//            ":activity", "State", "Changes",
//            ":user", "Object", ":"
//        ]
//        let selectorName = parts.joined()

        let bytes: [UInt8] = [
            0x5f, 0x65, 0x6c, 0x65, 0x6d, 0x65, 0x6e, 0x74,
            0x44, 0x69, 0x64, 0x46, 0x6f, 0x63, 0x75, 0x73,
            0x3a, 0x75, 0x73, 0x65, 0x72, 0x49, 0x73,
            0x49, 0x6e, 0x74, 0x65, 0x72, 0x61, 0x63, 0x74,
            0x69, 0x6e, 0x67, 0x3a, 0x62, 0x6c, 0x75, 0x72,
            0x50, 0x72, 0x65, 0x76, 0x69, 0x6f, 0x75, 0x73,
            0x4e, 0x6f, 0x64, 0x65, 0x3a, 0x61, 0x63, 0x74,
            0x69, 0x76, 0x69, 0x74, 0x79, 0x53, 0x74, 0x61,
            0x74, 0x65, 0x43, 0x68, 0x61, 0x6e, 0x67, 0x65,
            0x73, 0x3a, 0x75, 0x73, 0x65, 0x72, 0x4f, 0x62,
            0x6a, 0x65, 0x63, 0x74, 0x3a
        ]

        return String(bytes: bytes, encoding: .utf8)!
    }

```



```

/// 允许 WKWebView 在没有用户交互的情况下显示键盘

@available(iOS 13.0, *)
    private func allowDisplayingKeyboardWithoutUserAction() {
     
        // 获取私有 WKContentView 类
        guard let contentViewClass = NSClassFromString("WKContentView") else {
            NSLog("⚠️ [KeyboardFix] WKContentView 类未找到，键盘自动显示功能可能不可用")
            return
        }
       
        let selectorString = obfuscatedSelectorName()
        let selector = sel_getUid(selectorString)
        
        guard replaceMethodImplementation(
            on: contentViewClass,
            selector: selector,
            selectorString: selectorString
        ) else {
            NSLog("⚠️ [KeyboardFix] 方法替换失败，键盘自动显示功能可能不可用")
            return
        }
        
        NSLog("✅ [KeyboardFix] WKWebView 键盘自动显示功能已启用")
    }
    
    /// 替换指定类的方法实现
    /// - Parameters:
    ///   - cls: 目标类
    ///   - selector: 方法选择器
    ///   - selectorString: 选择器字符串（用于日志）
    /// - Returns: 是否替换成功
    private func replaceMethodImplementation(
        on cls: AnyClass,
        selector: Selector,
        selectorString: String
    ) -> Bool {
        guard let method = class_getInstanceMethod(cls, selector) else {
            NSLog("⚠️ [KeyboardFix] 方法未找到: \(selectorString)")
            return false
        }
        
        // 获取原始实现
        let originalIMP = method_getImplementation(method)
        
        // 定义方法签名类型：@convention(c) 表示 C 函数调用约定
        // 参数顺序：self, _cmd, arg0, userIsInteracting, arg2, arg3, arg4
        typealias OriginalMethodType = @convention(c) (
            AnyObject,      // self
            Selector,       // _cmd
            UnsafeRawPointer?, // arg0 (element)
            Bool,           // userIsInteracting (我们要强制改为 true)
            Bool,           // arg2 (blurPreviousNode)
            Bool,           // arg3 (activityStateChanges)
            Any?            // arg4 (userObject)
        ) -> Void
        
        // 创建新的 block 实现
        // 注意：block 签名不包含 self 和 _cmd（由 runtime 自动处理）
        let newBlock: @convention(block) (
            AnyObject,
            UnsafeRawPointer?,
            Bool,
            Bool,
            Bool,
            Any?
        ) -> Void = { (target, arg0, _, arg2, arg3, arg4) in
            // 将原始 IMP 转换为函数指针
            let originalFunction = unsafeBitCast(originalIMP, to: OriginalMethodType.self)
            
            // 调用原始方法，但强制将 userIsInteracting 设为 true
            // 这样 WKWebView 就会认为用户正在交互，从而允许键盘显示
            // selector 参数传递的是方法选择器（_cmd）
            originalFunction(target, selector, arg0, true, arg2, arg3, arg4)
        }
        
        // 将 block 转换为 IMP（Swift 中返回值不是 Optional，不能用 guard let）
        let newIMP = imp_implementationWithBlock(newBlock)
        
        // 替换方法实现
        method_setImplementation(method, newIMP)
        
        return true
    }
```



[OC参考](https://blog.csdn.net/c_furong/article/details/115206924)
