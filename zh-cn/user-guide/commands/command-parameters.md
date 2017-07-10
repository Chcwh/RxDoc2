# 命令参数

有时候，命令的执行逻辑需要参数。要实现这个目的，仅需要在创建 `ReactiveCommand` 的时候使用 `Create*` 的某个重载就可以了：

```cs
// 需要一个参数的同步命令
var command1 = ReactiveCommand.Create<int>(param => Console.WriteLine("Received parameter with type {0}: {1}.", param.GetType().Name, param);
// 输出 "Received parameter with type Int32: 42"
command1.Execute(42);

// 需要一个参数的异步命令
var command2 = ReactiveCommand.CreateFromObservable<int>(param => Observable.Return(param).Do(p => Console.WriteLine("Received parameter with type {0}: {1}.", p.GetType().Name, p)));
// 输出 "Received parameter with type Int32: 42"
command2.Execute(42);
```

参数的类型由 `ReactiveCommand<TParam, TResult>` 中的 `TParam` 指定。`command1` 和 `command2` 的类型都是 `ReactiveCommand<int, Unit>`。

通常，应该避免使用参数。最好是在视图模型中为命令执行逻辑需要的状态定义属性。

