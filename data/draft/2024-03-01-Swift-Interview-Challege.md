---
title: 'Swift Interview Challege'
date: '2024/03/01'
lastmod: '2024/03/01'
tags: ['Swift', 'Interview', 'Challenge']
draft: false
summary: '在这篇文章中，我们将讨论 Swift 中的一些常见面试问题，包括匹配括号、SceneDelegate 方法、闭包和内存地址等。'
images:
  [
    'https://assets-global.website-files.com/62ebf26875eba5d0775548fe/6445f9fc857e3615b7d1798f_Ios%20swift.webp',
  ]
authors: ['default']
layout: PostLayout
---

## Code Challenge

Given an array of strings, return an array of strings that indicate whether each string has matching braces.
If the string has matching braces, return "YES"; otherwise, return "NO".

```swift
func matchingBraces(braces: [String]) -> [String] {
    var result = [String]()
    for i in 0..<braces.count {
        var stack = [Character]()
        for char in brace {
            if char == "(" || char == "{" || char == "[" {
                stack.append(char)
            } else {
                if stack.isEmpty {
                    result.append("NO")
                    break
                }
                let last = stack.removeLast()
                if (char == ")" && last != "(") || (char == "}" && last != "{") || (char == "]" && last != "[") {
                    result.append("NO")
                    break
                }
            }
        }
        if stack.isEmpty {
            result.append("YES")
        } else {
            result.append("NO")
        }
    }
    return result
}
```

## Swift SceneDelegate Methods

在 Swift 中，SceneDelegate 类中的方法与应用程序的生命周期相关，用于管理和响应应用程序场景（scene）的状态变化。以下是你提到的方法的执行顺序，以及它们各自的作用：

scene(\_:willConnectTo:options:): 这个方法是场景生命周期中最先调用的方法，用于场景的初始设置和配置。当应用程序启动并且新的场景（session）被创建时，该方法被调用。在这里，你可以设置窗口和视图控制器等。

sceneWillEnterForeground(\_:): 当应用程序的场景即将从后台进入前台时（但还没有完全处于活跃状态之前），这个方法被调用。通常用于准备 UI 更新和场景恢复。

sceneDidBecomeActive(\_:): 当场景已经变成活跃状态时，这个方法被调用。场景变成活跃状态意味着用户可以与应用程序进行交互。在这个方法中，你可以启动任务，更新界面等。

总结执行顺序：

当应用启动并且场景被创建时，首先调用 scene(_:willConnectTo:options:)。
当应用即将从后台进入前台但尚未完全活跃时，调用 sceneWillEnterForeground(_:)。
当应用场景已经变成活跃状态，用户可以与之交互时，调用 sceneDidBecomeActive(\_:)。
这些方法允许你对应用程序的不同状态进行管理和响应，确保应用程序在各种场景下都能正确地运行和显示。

## Swift Closure

在 Swift 中，闭包（Closure）是一种自包含的功能代码块，可以在代码中被传递和使用。闭包可以捕获和存储其所在上下文中任意常量和变量的引用。Swift 为闭包提供了一种优雅的语法，使得编写和使用闭包变得非常简单和直观。

```swift
func makeIncrementer(incrementAmount: Int) -> () -> Int {
    var total = 0
    let incrementer: () -> Int = {
        total += incrementAmount
        return total
    }
    return incrementer
}

let incrementByTwo = makeIncrementer(incrementAmount: 2)
let result1 = incrementByTwo() // 调用一次，期望total增加2
let result2 = incrementByTwo() // 再调用一次，期望total再增加2
```

在上面的例子中，makeIncrementer(incrementAmount:) 是一个函数，它接受一个整数参数 incrementAmount，并返回一个闭包。这个闭包捕获了一个变量 total，然后返回 total 的值。每次调用 incrementByTwo() 时，total 的值都会增加 incrementAmount。这种方式可以实现一个简单的计数器功能。

## Swift 内存地址

```javascript
func getBufferAddress<T>(array: [T]) -> String {
    return array.withUnsafeBufferPointer { buffer in
        return String(reflecting: buffer.baseAddress)
    }
}

let flag1:Bool = (arr1==arr2)
let flag2:Bool = (getBufferAddress(array:arr1) == getBufferAddress(array:arr2))

arr2[0] = 100
arr1[0] =100

let flag3:Bool = (arr1==arr2)
let flag4:Bool = (getBufferAddress(array:arr1) == getBufferAddress(array:arr2))
```

flag1: 比较 arr1 和 arr2 的内容是否相等。在修改之前，arr1 和 arr2 的内容完全相同（都是 [1, 2, 3, 4]），因此 flag1 是 true。

flag2: 使用 getBufferAddress 函数比较 arr1 和 arr2 的内存地址。由于 Swift 数组采用 copy-on-write 机制，在修改前，arr1 和 arr2 应该指向同一块内存区域（假设没有进行任何修改操作），所以 flag2 也应该是 true。

然后，arr2[0] = 100 和 arr1[0] = 100 被执行，触发了 copy-on-write 机制，为每个数组创建了独立的内存拷贝。这意味着 arr1 和 arr2 现在指向不同的内存地址。

flag3: 再次比较 arr1 和 arr2 的内容。由于它们都被修改为 [100, 2, 3, 4]，内容上仍然相等，因此 flag3 是 true。

flag4: 使用 getBufferAddress 函数比较修改后的 arr1 和 arr2 的内存地址。由于 copy-on-write 机制确保当数组被修改时，它们拥有独立的内存地址，因此 flag4 是 false。

什么是 Swift 的 copy-on-write 机制？
