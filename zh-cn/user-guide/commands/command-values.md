# 返回值

命令的执行会产生返回值，其类型由 `ReactiveCommand<TParam, TResult>` 中的 `TResult` 指定。如果不需要返回值，那么将 `TResult` 指定为 `Unit`。实际上，在使用创建 `Create*` 重载创建没有返回值的命令时，会自动发生：

```cs
// 没有返回值的同步命令
var command1 = ReactiveCommand.Create(() => { });

// 基于可观察对象的，没有返回值的异步命令
var command2 = ReactiveCommand.CreateFromObservable(() => Observable.Return(Unit.Default));

// 基于Task，没有返回值的异步命令
var command3 = ReactiveCommand.CreateFromTask(async () => await Task.Delay(TimeSpan.FromSeconds(2)));
```

上面的所有命令的类型都是 `ReactiveCommand<Unit, Unit>` 。注意 `CreateFromObservable` 最终需要返回一个 `IObservable<T>`，因此必须指定 `T` 。这样的话，要用 `CreateFromObservable` 创建一个 `ReactiveCommand<Unit, Unit>` ，必须确保可观察对象的类型是 `IObservable<Unit>`。

如果想创建能够返回值的命令，只需要使用适当的 `Create*` 方法：

```cs
// 执行后总是返回 42 的同步命令
var command1 = ReactiveCommand.Create(() => 42);

// 基于可观察对象的，执行后总是返回 42 的同步命令
var command2 = ReactiveCommand.CreateFromObservable(() => Observable.Return(42));

// 基于Task，执行后总是返回 42 的同步命令
var command3 = ReactiveCommand.CreateFromTask(() => Task.FromResult(42));
```

这里，所有的命令的类型都是 `ReactiveCommand<Unit, int>`。订阅 `Execute` 返回的可观察对象将会 tick through 值 `42`。

