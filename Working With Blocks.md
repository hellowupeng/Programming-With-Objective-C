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

**Blocks 携带参数和返回值**

Blocks 也能像方法和函数一样携带参数和返回值。

举个例子，考虑一个变量引用一个返回两个值相乘的结果的 block：

```objective-c
double (^multiplyTwoValues)(double, double);
```

相应的 block 字面量可能看起来像这样：

```objective-c
^(double firstValue, double secondValue){
    return firstValue * secondValue;
}
```

firstValue 和 secondValue 被用于在 block 被调用时引用提供的值，就像任何函数定义一样。在这个例子里，返回类型从 block 里的返回语句推断出。

如果你愿意，你可以在脱字符和参数列表之间明确的指定返回类型：

```objective-c
^double(double firstValue, double secondValue){
    return firstValue * secondValue;
}
```

一旦你已经声明并定义了 block，你可以只像一个函数一样调用它：

```objective-c
double (^multiplyTwoValues)(double, double) = ^(double firstValue, double secondValue) {
    return firstValue * secondValue;
};

double result = multiplyTwoValues(2, 4);
NSLog(@"The result is %f", result);
```

**Blocks 能从封闭范围捕获值**

和包含可执行代码一样，block 也拥有从它的封闭范围捕获状态的能力。

如果你从方法里声明一个 block 字面量，例如，捕获那个方法范围里的任何可访问的值是可能的，像这样：

```objective-c
- (void)testMethod {
    int anInteger = 42;
    
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    }
    
    testBlock();
}
```

在这个例子里，anInteger 在 block 外面被声明，而在 block 被定义时值被捕获。

只是值被捕获，除非你另外指定。这意味了如果你在你定义 block 时和它被调用时期间改变变量的外部值，像这样：

```objective-c
int anInteger = 42;

void (^testBlock)(void) = ^{
    NSLog(@"Integer is: %i", anInteger);
}

anInteger = 84;

testBlock();
```

被 block 捕获的值不受影响。这意味着输出结果仍会显示：

```objective-c
Integer is: 42
```

他也意味着 block 不能改变原始变量的值，或甚至捕获的值（它作为一个 const 变量被捕获）。

**使用 __block 变量共享存储**

如果你需要能够改变从 block 里捕获变量的值，你可以在原始的变量声明上使用 `__block`存储类型修饰符。这意味着变量活在原始的变量的词法范围和任何在那个范围里被声明的 block 之间的存储被共享。

举个例子，你可以像这样重写之前的例子：

```objective-c
__block int anInteger = 42;

void (^testBlock)(void) = ^{
    NSLog(@"Integer is: %i", anInteger);
};

anInteger = 84;

testBlock();
```

由于 anInteger 被声明为一个 __block 变量，它的存储被和 block 声明共享。这意味着日志输出现在会显示：

```objective-c
Integer is: 84
```

它也意味着 block 能修改原始的值，像这样：

```objective-c
__block int anInteger = 42;

void (^testBlock)(void) = ^{
    NSLog(@"Integer is: %i", anInteger);
    anInteger = 100;
};

testBlock();
NSLog(@"Value of original variable is now: %i", anInteger);
```

这次，输出会显示：

```objective-c
Integer is: 42
Value of original variable is now: 100
```

**你可以传递 Blocks 作为参数到方法或函数**

在这个章节里前面的每个例子都在 block 被定义后立即调用。在实践中，通常传递 blocks 到函数或方法用于别处调用。你可以使用 Grand Central Dispatch 在后台调用 block，例如，或者定义一个表示一个会被重复调用的任务的 block，例如在枚举一个集合时。并发和枚举在这章后面覆盖。

Blocks 也被用于回调，定义在任务完成时会被执行的代码。例如，你的应用可能需要通过创建执行复杂任务的对象来响应用户动作，例如从 web 服务请求信息。由于任务可能需要很长时间，你应该在任务发生时显示某种进度指示器，然后一旦任务被完成就隐藏指示器。

也可以使用代理来完成这个：你需要创建一个合适的代理协议，实现必要的方法，设置你的对象作为任务的代理，然后等待它一旦任务完成就在你的对象上调用一个代理方法。

Blocks 使这变得更容易，然而，因为你能在你初始化任务时定义回调行为，像这样：

```objective-c
- (IBAction)fetchRemoteInformation:(id)sender {
    [self showProgressIndicator];
    
    XYZWebTask *task = ...
        
    [task beginTaskWithCallbackBlock:^{
    	[self hideProgressIndicator];
    }];
}
```

这个例子调用一个方法显示进度指示器，然后创建任务并告诉它开始。回调 block 指定一旦任务完成就会被执行的代码；在这种情况下，它仅仅调用一个方法来隐藏进度指示器。注意这个回调 block 捕获了 self 以便能够在被调用时调用 hideProgressIndicator 方法。在捕获 self 时小心些很重要，因为它容易创建一个强引用循环，如稍后在 Avoid Strong Reference Cycles when Capturing self 所述。

就代码可读性而言，block 使在一个地方准确查看在任务完成之前和之后发生什么变得容易，避免需要通过代理方法跟踪来找出会发生什么事情。

在这个例子里显示的 beginTaskWithCallbackBlock: 方法的声明看起来像这样：

```objective-c
- (void)beginTaskWithCallbackBlock:(void (^)(void))callback;
```

(void (^)(void)) 指定参数是一个不携带任何参数或返回值得 block。这个方法的实现能用通常的方式调用 block。

```objective-c
- (void)beginTaskWithCallbackBlock:(void (^)(void))callback {
    ...
    callback();
}
```

期望一个拥有一个或多个参数的 block 被用和一个 block 变量相同的方法指定的方法参数：

```objective-c
- (void)doSomethingWithBlock:(void (^)(double, double))block
{
    ...
    block(21.0, 2.0);
}
```

**Block 总应该是方法的最后一个参数**

一个方法只使用一个 block 参数是最佳实践。如果方法也需要其他非 block 参数，block 应该放在末尾：

```objective-c
- (void)beginTaskWithName:(NSString *)name completion:(void (^)(void))callback;
```

这使得在内联指定 block 时方法调用变得更易于阅读，像这样：

```objective-c
[self beginTaskWithName:@"MyTask" completion:^{
    NSLog(@"The task is complete");
}];
```

**使用类型定义简化 Block 语法**

###### Blocks 能简化枚举

###### Blocks 能简化并发任务