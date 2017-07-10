# UI 线程和调度器

确保总是在 `RxApp.MainThreadScheduler` 上更新 UI，以保证 UI 在 UI 线程上产生变化。在实践中，就是确保在主线程调度器上更新视图模型。

## 可以
```csharp
FetchStuffAsync()
  .ObserveOn(RxApp.MainThreadScheduler)
  .Subscribe(x => this.SomeViewModelProperty = x);
```

## 更好

更好的是，将调度程序传递给异步操作 - 这对于更复杂的任务通常是必需的。

```csharp
FetchStuffAsync(RxApp.MainThreadScheduler)
  .Subscribe(x => this.SomeViewModelProperty = x);
```

## 不可以
```csharp
FetchStuffAsync()
  .Subscribe(x => this.SomeViewModelProperty = x);
```

