# 控制可执行性

响应命令在执行过程中自动变得不可用。这是因为，`CanExecute` 这个可观察对象在执行开始时会 tick `false` ，在完成后 tick `true`。但是，有时候可能需要进一步控制命令的可执行性。比如，可能需要在用户没有输入用户名和密码之前禁用登陆命令。

要实现这个目标，可以在创建命令的时候为 `canExecute` 参数传递一个 `IObservable<bool>`：

```cs
var canExecute = this
    .WhenAnyValue(
        x => x.UserName,
        x => x.Password,
        (u, p) => !string.IsNullOrEmpty(u) && !string.IsNullOrEmpty(p));
var command = ReactiveCommand.CreateFromObservable(this.LogOnAsync, canExecute);
```

这里， `command` 在可执行性上有一个附加条件，即必须提供 `UserName` 和 `Password`。当然，`command` 在 `LogOnAsync` 执行期间依然不可用，即使 `UserName` 和 `Password` 都有效。换句话说，`canExecute` *补充了* 默认的可执行性行为，而不是替换。

> **警告** 出于性能原因，`ReactiveCommand` 不会将 `canExecute` 可观察对象合并到主线程。 如果想 `canExecute` 可观察对象在主线程上 tick，需要添加一个 `ObserveOn` 调用。
