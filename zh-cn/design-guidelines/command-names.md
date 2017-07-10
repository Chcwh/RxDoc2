# 命令名称

不要为 `ReactiveCommand` 属性加上 `Command` 后缀；而是使用一个能够描述命令动作的动词命名属性。比如说：

```csharp
public ReactiveCommand Synchronize { get; private set; }

// 在构造函数里面：
Synchronize = ReactiveCommand.CreateAsyncObservable(
  _ => SynchronizeImpl(mergeInsteadOfRebase: !IsAhead));

```

在 `ReactiveCommand` 的实现太长或太复杂，不适合作为匿名委托的时候，为实现的方法取其命令相同的名称，但是加上 `Impl` 后缀（例如上面的 `SychronizeImpl`）。
