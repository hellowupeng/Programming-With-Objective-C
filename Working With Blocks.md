## Working With Blocks

一个 Objective-C 类定义了一个连接数据和相关行为的对象。有时候，它只对表示一个单一任务或行为单位有意义，而不是一个方法集合。

Blocks 是一个添加到 C、Objective-C 和 C++ 的语言级特性，他允许你创建能被传入方法或函数似乎是值的清晰代码片段。Blocks 是 Objective-C 对象，意味着他们能被添加到像 NSArray 或 NSDictionary 一样的结合。它们也有能力从封闭范围捕获值，使它们类似于其他编程语言里的闭包（closures）或 lambdas 表达式。

这个章节说明了声明和引用 blocks 的语法，并展示了如何使用 blocks 简化常见的任务，例如集合、枚举。更多信息，查看 Blocks Programming Topics。

###### Block 语法

使用脱字符（^）定义一个 block 字面量的语法，像这样：

```objective-c
^{
    NSLog(@"This is a block");
}
```

和函数和方法定义一样，大括号标示了 block 的开始和结束。在这个示例里，block 不返回任何值，不携带任何参数。

同样你能使用一个函数指针来引用一个 C 函数，你能声明一个变量来跟踪 block，像这样：

```objective-c
void (^simpleBlock)(void);
```

如果你不习惯于处理 C 函数指针，这种语法可能看起来有些不正常。这个示例声明一个叫做 `simpleBlock` 的变量来引用一个不携带参数并且没有返回值的 block，意味着这个变量能被分配给在上面展示的 block 字面量，像这样：

```objective-c
void (^simpleBlock)(void) = ^{
    NSLog(@"This is a block");
};
```

一旦你已经声明并分配了一个 block 变量，你可以使用它来调用这个 block：

```objective-c
simpleBlock();
```

> 注意：如果你尝试使用一个未分配的变量（nil block 变量）调用 block，应用会崩溃。

###### Blocks 能简化枚举

###### Blocks 能简化并发任务