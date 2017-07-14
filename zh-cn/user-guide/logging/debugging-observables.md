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
