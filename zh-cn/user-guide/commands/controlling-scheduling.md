# Controlling Scheduling

默认情况下，`ReactiveCommand` 使用 `RxApp.MainThreadScheduler` 来调度事件。包括 `CanExecute`， `IsExecuting`， `ThrownExceptions` 的值和命令自身的返回值。通常 UI 组件会订阅这些可观察对象，因此这是合理的。但是，在为视图模型编写单元测试的时候，可能需要在调度方面更多的控制权。

> **警告** 重要的是，要知道响应命令的执行逻辑 `不一定` 会被调度到所提供的调度器上面执行（just as is the case for any `canExecute` observable you provide）。Instead, it is left to the caller to implement any required scheduling inside their execution pipeline. This means it is entirely possible for your execution logic to execute on a thread other than that owned by the provided scheduler:
>
> ```cs
> var command = ReactiveCommand.Create(() => Console.WriteLine(Environment.CurrentManagedThreadId), outputScheduler: RxApp.MainThreadScheduler);
>
> // this will output the ID of the thread from which you make this call, not necessarily the ID of the main thread!
> command.Execute().Subscribe();
> ```

所有的 `Create*` 方法都有一个可选的 `outputScheduler` 参数，因此在需要的情况下可以传入一个自定义调度器：

```cs
var command = ReactiveCommand.Create(() => {}, outputScheduler: someScheduler);
```

> **注意** 如果在测试中使用 ReactiveUI 的 `With` 扩展方法，可以创建使用默认调度行为的命令。这是因为 `With` 扩展方法会用 `RxApp.MainThreadScheduler` 替换你提供的调度器。
