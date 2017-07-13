## 取消

如果命令的执行需要很长的时间，允许执行可取消会很有用。取消支持可以仅用于视图模型内部，或者公开给用户。

### 基本方式

最主要的方式，是通过销毁执行订阅取消命令的执行：

```cs
var subscription = someReactiveCommand
    .Execute()
    .Subscribe();

// 这会取消命令的执行
subscription.Dispose();
```

但是，这需要取得并保存订阅。如果使用绑定执行命令，那就没办法了。

### 通过另一个可观察对象取消

Rx 支持在另一个可观察对象 tick 的时候取消一个可观察对象。通过使用 `TakeUntil` 操作符：

```cs
var cancel = new Subject<Unit>();
var command = ReactiveCommand
    .CreateFromObservable(
        () => Observable
            .Return(Unit.Default)
            .Delay(TimeSpan.FromSeconds(3))
            .TakeUntil(cancel));

// somewhere else
command.Execute().Subscribe();

// 取消上面的执行
cancel.OnNext(Unit.Default);
```

当然，可能不想就为了取消就创建 Subject。通常是已经有一个想用作取消信号的可观察对象了。典型的例子是有一个命令来取消另一个：

```cs
public class SomeViewModel : ReactiveObject
{
    public SomeViewModel()
    {
        this.CancelableCommand = ReactiveCommand
            .CreateFromObservable(
                () => Observable
                    .Return(Unit.Default)
                    .Delay(TimeSpan.FromSeconds(3))
                    .TakeUntil(this.CancelCommand));
        this.CancelCommand = ReactiveCommand.Create(
            () => { },
            this.CancellableCommand.IsExecuting);
    }

    public ReactiveCommand<Unit, Unit> CancelableCommand
    {
        get;
        private set;
    }

    public ReactiveCommand<Unit, Unit> CancelCommand
    {
        get;
        private set;
    }
}
```

这里我们有一个视图模型，其中有一个命令 `CancelableCommand`，可以通过执行另一个命令 `CancelCommand` 来取消。 注意在 `CancelableCommand` 正在执行时，`CancelCommand` 才能执行。

> **注意** 乍看之下，可能在 `CancelableCommand` 和 `CancelCommand` 之间可能会出现一个无法解决的循环依赖关系。 但是，请注意，`CancelableCommand` 在执行之前不需要解析其执行管道。 因此，只要 `CancelCommand` 在执行 `CancelableCommand` 之前就存在，循环依赖关系就被解决了。

### 使用 TPL 来取消

TPL 中的取消由 `CancellationToken` 和 `CancellationTokenSource` 处理。 提供 TPL 集成的 Rx 运算符通常会有重载，允许在创建任务的时候传递一个 `CancellationToken`。 这些重载的想法是，如果订阅被销毁，您收到的 `CancellationToken` 将被取消。 所以你应该将令牌传递给所有相关的异步操作。 `ReactiveCommand` 为 `CreateFromTask` 提供了类似的重载。

请看例子：

```cs
public class SomeViewModel : ReactiveObject
{
    public SomeViewModel()
    {
        this.CancelableCommand = ReactiveCommand
            .CreateFromTask(
                ct => this.DoSomethingAsync(ct));
    }

    public ReactiveCommand<Unit, Unit> CancelableCommand
    {
        get;
        private set;
    }

    private async Task DoSomethingAsync(CancellationToken ct)
    {
        await Task.Delay(TimeSpan.FromSeconds(3), ct);
    }
}
```

有几个重点需要注意：

1. `DoSomethingAsync` 方法需要一个 `CancellationToken`
2. 该令牌传递给 `Task.Delay` ，因此如果令牌被取消的话，延迟会提前结束。
3. 有一个 `CreateFromTask` 重载，因此可以给 `DoSomethingAsync` 传递令牌。该令牌会在执行订阅被销毁的时候自动取消。（自动创建令牌？）

上面的代码还可以这样：

```cs
var subscription = viewModel
    .CancelableCommand
    .Execute()
    .Subscribe();

// 这会取消命令的执行
subscription.Dispose();
```

但是，如果我们想要根据外部因素取消执行，就像我们对可观察性一样？ 既然我们只能访问 `CancellationToken` 而不是 `CancellationTokenSource`，那么我们如何才能实现，这一点并不明显。

除了完全使用 TPL（如果可能的话，建议，但不总是实用），实际上有很多方法可以实现。 也许最简单的方法是使用 `CreateFromObservable`：

```cs
public class SomeViewModel : ReactiveObject
{
    public SomeViewModel()
    {
        this.CancelableCommand = ReactiveCommand
            .CreateFromObservable(
                () => Observable
                    .StartAsync(ct => this.DoSomethingAsync(ct))
                    .TakeUntil(this.CancelCommand));
        this.CancelCommand = ReactiveCommand.Create(
            () => { },
            this.CancellableCommand.IsExecuting);
    }

    public ReactiveCommand<Unit, Unit> CancelableCommand
    {
        get;
        private set;
    }

    public ReactiveCommand<Unit, Unit> CancelCommand
    {
        get;
        private set;
    }

    private async Task DoSomethingAsync(CancellationToken ct)
    {
        await Task.Delay(TimeSpan.FromSeconds(3), ct);
    }
}
```

这种方法允许我们使用与上述纯 Rx 解决方案完全相同的技术。 区别在于我们的可观察流水线包含执行基于 TPL 的异步代码。

