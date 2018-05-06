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

如果你需要用相同的签名定义不止一个 block，你可能想要为那个签名定义你自己的类型。

举个例子，你可以为一个不携带参数或返回值的简单 block 定义一种类型，像这样：

```objective-c
typedef void (^XYZSimpleBlock)(void);
```

你然后能为方法参数或在创建 block 变量时使用你的自定义类型：

```objective-c
XYZSimple anotherBlock = ^{
    ...
};
```

```objective-c
- (void)beginFetchWithCallbackBlock:(XYZSimpleBlock)callback {
    ...
    callbackBlock();
};
```

自定义类型定义在处理返回 blocks 或携带其他 blocks 作为参数时特别有用。考虑以下例子：

```objective-c
void (^(^complexBlock)(void (^)(void))(void)) = ^(void (^aBlock)(void)) {
    ...
    return ^{
        ...
    };
};
```

complexBlock 变量引用一个携带另一个 block 作为一个参数（aBlock）和返回另一个 block 的 block。

重写代码使用类型定义使这变得更加可读：

```objective-c
XYZSimpleBlock (^betterBlock)(XYZSimpleBlock) = ^(XYZSimpleBlock aBlock) {
    ...
    return ^{
    	...    
    };
};
```

**对象使用属性跟踪 Blocks**

定义属性跟踪 block 的语法类似 block 变量：

```objective-c
@interface XYZObject : NSObject
@property (copy) void (^blockProperty)(void);
@end
```

> 注意：你应该指定 copy 作为属性特性，因为 block 需要被复制来在原始范围外跟踪它捕获的状态。在使用 Automatic Reference Counting 时这不是你需要关心的事情，因为它会自动地发生，但它对于属性特性显示组合的行为是最佳实践。更多信息，查看 Blocks Programming To

block 属性像任何其他 block 变量一样设置或调用：

```objective-c
self.blockProperty = ^{
    ...
};
self.blockProperty();
```

使用类型定义作为 block 属性声明也是可能的，像这样：

```objective-c
typedef void (^XYZSimpleBlock)(void);

@interface XYZObject : NSObject
@propery (copy) XYZSimpleBlock blockProperty;
@end
```

**在捕获 self 时避免强引用循环**

如果你需要在 block 里捕获 self，例如在定义一个回调 block 时，考虑内存管理意义很重要。

Blocks 保留任何捕获的对象的强引用，包括 self，意味着很容易造成强引用循环，例如，一个为一个捕获 self 的 block 保留一个 copy 属性的对象：

```objective-c
@interface XYZBlockKeeper : NSObject
@property (copy) void (^block)(void);
@end
```

```objective-c
@implementation XYZBlockKeeper
- (void)configureBlock {
    self.block = ^{
        [self doSomething]; // capturing a strong reference to self
							// creates a strong reference cycle
    };
}
...
@end
```

对于像这样的简单例子编译器会警告你，但是更复杂的例子可能在对象之间涉及多个强引用来创建循环，使诊断变得更复杂。

为了避免这个问题，捕获 self 的弱引用是最佳实践，像这样：

```objective-c
- (void)configureBlock {
    XYZBlockKeeper * __weak weakSelf = self;
    self.block = ^{
        [weakSelf doSomething]; // capture the weak reference to 								// avoid the reference cycle
    };
}
```

通过捕获指向 self 的弱指针，block 不会保留 XYZBlockKeeper 对象的强引用关系。如果那个对象在 block 被调用之前被释放，weakSelf 指针仅会被设置为 nil。

###### Blocks 能简化枚举

除了一般的完成处理器，许多 Cocoa 和 Cocoa Touch API 使用 blocks 简化常用任务，例如集合枚举。NSArray 类，例如，提供三个基于 block 的方法，包括：

```objective-c
- (void)enumerateObjectsUsingBlock:(void (^)(id obj, NSUInteger idx, BOOL *stop))block;
```

这个方法携带一个参数，是一个数组里每个项目都会调用一次的 block：

```objective-c
NSArray *array = ...
    [array enumerateObjectUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        NSLog(@"Object at index %lu is %@", idx, obj);
    }];
```

这个 block 自己携带三个参数，前两个引用数组里的当前对象和它的索引。第三个参数是一个指向一个你可以用于停止枚举的布尔变量，像这样：

```objective-c
[array enumerateObjectsUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
    if (...) {
        *stop = YES;
    }
}];
```

也可以通过使用 enumerateObjectsWithOptions:usingBlock: 方法来自定义枚举。指定 NSEnumerationReverse 选项，例如，会从相反的顺序遍历数组。

如果枚举 block 里的代码是处理器密集型并且对于并发执行是安全的，你可以使用 NSEnumerationConcurrent 选项：

```objective-c
[array enumerateObjectsWithOptions:NSEnumerationConcurrent usingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
    ...
}];
```

这个标记指示枚举 block 调用可以分发到多个线程，如果 block 代码尤其是处理器密集型的化，会提供一个潜在的性能提升。注意在使用这个选项时，枚举顺序是未定义的。

NSDictionary 类也提供基于 block 的方法，包括：

```objective-c
NSDictionary *dictionary = ...
    [dictionary enumerateKeysAndObjectsUsingBlock:^ (id key, id obj, BOOL *stop) {
        NSLog(@"key: %@, value: %@", key, obj);
    }];
```

这使枚举每个键值对比在使用传统的循环时更加方便。

###### Blocks 能简化并发任务

block 表示不同的工作单元，用从周围范围里捕获的可选的状态连接可执行代码。这使他对于使用一个 OSX 和 iOS 可用的并发选项的异步调用是理想的。而不是必须弄清楚如何同像线程一样的低级别的机制一起工作，你可以只使用 blocks 定义你的任务然后让系统作为处理器资源执行这些任务变得可用。

OSX 和 iOS 提供了各种并发技术，包括两种任务调度机制：操作队列和 GCD。这些机制围绕等待被调用的任务的队列的想法。你添加你的 blocks 到一个队列按你需要它们被调用的顺序，系统在处理器时间和资源变得可用时把它们移除队里用于调用。

串行队列一次只允许执行一个任务，队列里的下一个任务不会被移出队列并调用直到之前的任务已经完成。并发队列调用尽可能多的任务，不需要等待之前的任务完成。

**和操作队列一起使用 Block 操作**

操作队列是 Cocoa 和 Cocoa Touch 进行任务调度的途径。你创建一个 NSOperation 实例来封装一个工作单位以及任何必要的数据，然后添加那个操作到一个 NSOperationQueue 用于执行。

尽管你能创建你自己的自定义 NSOperation 子类来实现复杂任务，也可以使用 NSBlockOperation 来使用 block 创建一个操作，像这样：

```objective-c
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    ...
}];
```

也可以手动执行一个 operation，但是 operations 通常被添加到一个存在的操作队里或你自己创建的队列，准备执行：

```objective-c
// schedule task on main queue:
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
[mainQueue addOperation:operation];

// schedule task on background queue
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation];
```

如果你使用一个操作队列，你可以在队列之间配置优先级或依赖，例如指定一个操作不应该被执行直到一组其他操作已经完成。你也可以通过键值观察监听你的操作的状态变化，例如，使在任务完成时更新进度指示器变得更容易。

操作和操作队列的更多信息，查看 Operation Queues。

**和 GCD 一起 在调度队列上安排 Blocks**

如果你需要安排一个随意的代码块用于执行，你可以直接和由 GCD 控制的调度队列（dispatch queue）一起工作。调度队列使用期望的调用者同步或者异步地执行任务变得容易，并且采用先进先出的顺序执行它们的任务。

你可以创建你自己的调度队列或使用由 GCD 自动提供的调度队列之一。如果你需要安排一个用于同时执行的任务，例如，你可以通过使用 dispatch_get_global_queue() 函数获取一个存在队列的引用并指定队列优先级，像这样：

```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

要调度 block 到队列里，你使用 dispatch_async() 或 dispatch_sync() 函数。dispatch_async() 函数立即返回，无需等待 block 被调用。

```objective-c
dispatch_async(queue, ^{
    NSLog(@"Block for asynchronous execution");
});
```

dispatch_sync() 函数不返回直到 block 已经完成执行；你可以在并发 block 在继续之前需要在主线程上等待另一个任务完成的情况使用它。

调度队列和 GCD 的更多信息，查看 Dispatch Queues。