## 异步 VS 同步

尽管 `ReactiveCommand` 提供的 API 是异步的，也不需要异步执行执行逻辑。 如果您的命令不是 CPU 密集型或基于 I/O，则提供同步执行逻辑可能是有意义的。可以通过 `ReactiveCommand.Create` 创建命令：

```cs
var command = ReactiveCommand.Create(() => Console.WriteLine("a synchronous reactive command));
```

有几个 `Create` 能够创建需要参数或能返回值的命令。这将在后文详细讨论。

另一方面，如果命令逻辑是 CPU 或 I/O 阻塞的话，需要使用 `CreateFromObservable` 或 `CreateFromTask`：

```cs
// 这里使用可观察对象模拟异步
var command1 = ReactiveCommand.CreateFromObservable(() => Observable.Return(Unit.Default).Delay(TimeSpan.FromSeconds(3)));

// 这里使用 TPL 模拟异步
var command2 = ReactiveCommand.CreateFromTask(async () =>
    {
        await Task.Delay(TimeSpan.FromSeconds(3)); 
    });
```

同样，这两个方法也有重载可以创建需要参数或能返回值的命令。

无论命令是同步还是异步的，都可以通过 `Execute` 来执行。然后获得一个能在执行完成后 tick 命令结果的可观察对象。同步命令将会立即执行，因此获得的可观察对象已被完成。返回的可观察对象是 behavioral though，因此这之后订阅也能收到返回值。

> **警告** 像 Rx 通常的一样，`Execute1 返回的可观察对象是冷的。 也就是说，除非有人订阅，否则什么都不会发生。 这种订阅通常是由绑定基础设施完成的。 但是在直接调用 `Execute` 的情况下，记住它很懒惰是非常重要的。
