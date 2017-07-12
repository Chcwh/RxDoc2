# 调用命令

有时候想方便的在没有观察到用户交互的情况下执行一命令。比如，通过每5分钟自动执行一次 `SaveCommand` 。`InvokeCommand` 扩展可以轻易的实现：

```cs
var interval = TimeSpan.FromMinutes(5);
Observable
    .Timer(interval, interval)
    .InvokeCommand(this.ViewModel, x => x.SaveCommand);
```

> **提示** `InvokeCommand` 遵循命令的可执行性。就是说，如果命令的 `CanExecute` 返回 `false`，`InvokeCommand` 不会在可观察对象 tick 的时候执行命令。
