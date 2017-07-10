# 使用可观察属性帮助器而不是直接设置属性

当某个属性的值依赖于一个或多个其它属性，或者可观察流时，与其直接设置值，不如在可能的情况下使用 `ObservableAsPropertyHelper` 和 `WhenAny`。

## 可以

```csharp
public class RepositoryViewModel : ReactiveObject
{
  public RepositoryViewModel()
  {
    canDoIt = this.WhenAny(x => x.StuffFetched, y => y.OtherStuffNotBusy, (x, y) => x && y)
      .ToProperty(this, x => x.CanDoIt);
  }

  readonly ObservableAsPropertyHelper<bool> canDoIt;
  public bool CanDoIt
  {
    get { return canDoIt.Value; }  
  }	
}
```

## 不可以

```csharp
this.WhenAny(x => x.StuffFetched, y => y.OtherStuffNotBusy, (x, y) => x && y)
  .ObserveOn(RxApp.MainThreadScheduler)
  .Subscribe(x => CanDoIt = x);
```

## 为什么？

 - `ObservableAsPropertyHelper` 负责触发 `INotifyPropertyChanged` 事件 - 如果创建只读属性的话，会少写很多代码。
 - `WhenAny` 允许组合多个属性，将其变化形成可观察流，并生成视图模型特定输出。