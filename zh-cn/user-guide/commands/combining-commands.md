## 组合命令

有时可以将多个命令聚合成一个。 例如，假如有一个允许用户清除各个缓存（浏览历史记录，下载历史记录，Cookie）或清除所有缓存的浏览器。 将有一个清除每个单独缓存的命令，每个缓存可能有自己的逻辑来指示命令的可执行性。 对于清除所有缓存的命令来说，重复或组合所有这些逻辑，不仅重复还容易出错。 组合命令提供了解决这种情况的优雅手段：

```cs
IObservable<bool> canClearBrowsingHistory = ...;
var clearBrowsingHistoryCommand = ReactiveCommand.CreateFromObservable(
    this.ClearBrowsingHistory,
    canClearBrowsingHistory);

IObservable<bool> canClearDownloadHistory = ...;
var clearDownloadHistoryCommand = ReactiveCommand.CreateFromObservable(
    this.ClearDownloadHistory,
    canClearDownloadHistory);

IObservable<bool> canClearCookies = ...;
var clearCookiesCommand = ReactiveCommand.CreateFromObservable(
    this.ClearCookies,
    canClearCookies);

// 组合所有命令
var clearAllCommand = ReactiveCommand
    .CreateCombined(
        new [] { clearBrowsingHistoryCommand, clearDownloadHistoryCommand, clearAllCommand });
```

组合命令的可执行性取决于所有子命令的可执行性。就是说，如果任何一个子命令现在不能执行，那么组合命令也不能。另外，也可以添加额外的可执行性逻辑：

```cs
IObservable<bool> canClearAll = ...;
var clearAllCommand = ReactiveCommand
    .CreateCombined(
        new [] { clearBrowsingHistoryCommand, clearDownloadHistoryCommand, clearAllCommand },
        canClearAll);
```

在这种情况下，`clearAllCommand` 只会在所有子命令可执行且 `canClearAll` 的最新值为 `true` 的时候才能执行。

> **提示** `CreateCombined` 方法的所有命令的类型必须一致。不能组合诸如 `ReactiveCommand<Unit, Unit>` 与 `ReactiveCommand<int, Unit>`，`ReactiveCommand<Unit, Unit>` 与 `ReactiveCommand<Unit, int>`。这是因为所有的命令都会收到提供给组合命令的参数，且组合命令的返回值是所有子命令返回值的列表。



