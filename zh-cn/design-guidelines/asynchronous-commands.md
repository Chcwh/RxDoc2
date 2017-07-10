# 异步命令

建议在所有情况下使用异步 `ReactiveCommand` 替代更基础的 `ReactiveCommand` 命令，除了最简单的任务。在 ReactiveUI 中，不应该将业务代码放在订阅块中 - 订阅仅用于记录操作结果，或关联属性与属性。

## 应该

```csharp
// In XAML
<Button Command="{Binding Delete}" .../>

public class RepositoryViewModel : ReactiveObject
{
  public RepositoryViewModel() 
  {
    Delete = ReactiveCommand.CreateAsyncObservable(x => DeleteImpl());
    Delete.IsExecuting.ToProperty(this, x => x.IsDeleting, out _isDeleting);
    Delete.ThrownExceptions.Subscribe(ex => this.Log().ErrorException("Something went wrong", ex));
  }

  public ReactiveCommand<Unit> Delete { get; private set; }

  readonly ObservableAsPropertyHelper<bool> _isDeleting;
  public bool IsDeleting { get { return _isDeleting.Value; } }

  public IObservable<Unit> DeleteImpl()
  {
    return Observable.Start(() => /* ... */);
  }
}
```

## 不应该

```csharp
// In XAML
<Button Command="{Binding Delete}" .../>

public class RepositoryViewModel : ReactiveObject
{
  public RepositoryViewModel() 
  {
    Delete = ReactiveCommand.Create();
    // This will block the UI thread while DeleteImpl runs
    Delete.Subscribe(async _ => await DeleteImpl());
    // These will not do what you expect
    Delete.IsExecuting.ToProperty(this, x => x.IsDeleting, out _isDeleting);
    Delete.ThrownExceptions.Subscribe(ex => this.Log().ErrorException("Something went wrong", ex));
  }

  public ReactiveCommand<object> Delete { get; private set; }

  readonly ObservableAsPropertyHelper<bool> _isDeleting;
  public bool IsDeleting { get { return _isDeleting.Value; } }

  public IObservable<Unit> DeleteImpl()
  {
    return Observable.Start(() => /* ... */);
  }
}
```

## 为什么？

许多 `ReactiveCommand` 的功能都在异步版本中。基础版本中以下特性与预期不同：

* `IsExecuting`  observable will not report on your asynchronous method when it is inside the `Subscribe`
* `ThrownExceptions` 不会捕获任何东西
* `CanExecute` 命令在执行的时候没有变化，所以有可能同时执行多个。



