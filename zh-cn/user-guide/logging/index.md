# Logging


# Why does Splat exist?
> https://github.com/paulcbetts/splat/issues/46#issuecomment-56550457
>
> One thing that motivates me to write my own instead of using the legion of others, is that most loggers give zero thought to perf concerns on mobile devices - they're all written for servers, so none of them think about CPU perf or allocations. The best imho is Serilog, but it allocates way too much stuff imho to be usable on mobile

# Conversations from Slack  

    haacked [8:56 AM] So does rxui6 get rid of this.Log()?

    paulcbetts [8:57 AM] No, it's part of Splat

    paulcbetts [8:57 AM] It just got moved

    haacked [8:59 AM] But there's no implementation for nlog yet.

    paulcbetts [9:00 AM] Correct - you should be able to copy-paste the
    reactiveui-nlog version though

    paulcbetts [9:01 AM] Like, the code is exactly the same, it's just in a
    different assembly

    haacked [9:01 AM] BTW, this.Log() is a static method. So it's effectively the
    same thing. :wink:

    haacked [9:04 AM] I guess the benefit is you don't have to define a static
    variable in every class, which is nice.

    haacked [9:04 AM] Does it somehow use the class defined by `this` to create the
    scope of the logger? So each class still gets its own?

    paulcbetts [9:04 AM] Yeah

    paulcbetts [9:05 AM] That's the scam, is that the `this` is used to set the
    class name for the logger

    haacked [9:07 AM] But I assume every call to `this.Log()` doesn't create a new
    logger. Instead, you look it up based on the class name in some concurrent
    dictionary?

    paulcbetts [9:08 AM] It's stored in a MemoizedMRUCache as I recall

    paulcbetts [9:09 AM] Can't remember the details

    haacked [9:10 AM] :cool: thanks!

# 日志

ReactiveUI 也有自己的日志框架，可以用来调试你的程序和 ReactiveUI。你可能会问，“真糟糕，又是一个日志框架？”。RxUI 自己实现日志框架是为了移植性——没有一个主流通用的日志框架支持所有 ReactiveUI 支持的平台，并且许多是面向服务的框架且不适合用于简单的移动应用程序的日志记录。

### this.Log() 和 IEnableLogger

ReactiveUI 的记录器用起来有一点麻烦，和其他框架相比 —— 它的思路来自于 Rails 的记录器。要使用它，需要让你的类实现 `IEnableLogger` 接口：

```cs
public class MyClass : IEnableLogger
{
    // IEnableLogger 实际上不需要实现任何东西
}
```

现在，你可以在你的类上调用 `Log` 方法了。由于扩展方法的工作机制，你必须前置一个 `this`：

```cs
this.Log().Info("Downloaded {0} tweets", tweets.Count);
```

日志有**五个**级别： `Debug`， `Info`， `Warn`， `Error`，和 `Fatal`。此外，还有一些特殊的方法用于记录异常 —— 比如，`this.Log().InfoException(ex, "Failed to post the message")`。

这招不能在静态方法中使用，必须使用另一个备用的方法，`LogHost.Default.Info(...)`。

### 调试 Observable

ReactiveUI 有一些帮助类用于调试 IObservable。最简单的方法是 `Log`，可以记录 Observable 发生的异常：

```cs
// 注意: 由于 Log 实际上和 Rx 操作符如 Select 或 Where 类似，
// 所以在被订阅之前不会记录任何东西。
this.WhenAny(x => x.Name, x => x.Value)
    .SelectMany(async x => GoogleForTheName(x))
    .Log(this, "Result of Search")
    .Subscribe();
```

另一个用于调试 Observable 的方法是 `LoggedCatch`。这个方法做的工作和 Rx 的 `Catch` 操作符一样，除此之外还记录异常到记录器。例如：

```cs
var userAvatar = await FetchUserAvatar()
    .LoggedCatch(this, Observable.Return(default(Avatar)));
```

### 配置记录器

要配置记录器，注册一个 `ILogger` 的实现（有一些内置的实现，比如 `DebugLogger`）。这个例子演示了如何以自定义级别使用内置记录器：

```cs
// 只需要记录错误
var logger = new DebugLogger() { LogLevel = LogLevel.Error };
RxApp.MutableResolver.RegisterConstant(logger, typeof(ILogger));
```

如果真的需要控制日志记录的方式，需要实现 `IFullLogger`，其允许你控制每个日志重载。
