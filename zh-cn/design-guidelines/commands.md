# 命令

为用户界面绑定命令而不是方法

## 应该

```csharp
// In XAML
<Button Command="{Binding Delete}" .../>

public class RepositoryViewModel : ReactiveObject
{
  public RepositoryViewModel() 
  {
    Delete = ReactiveCommand.CreateAsyncObservable(x => DeleteImpl());
    Delete.ThrownExceptions.Subscribe(ex => /*...*/);
  }

  public ReactiveAsyncCommand Delete { get; private set; }

  public IObservable<Unit> DeleteImpl() {...}
}
```

## 不应该

使用 Caliburn.Micro 约定关联按钮和命令：

```csharp
// In XAML
<Button x:Name="Delete" .../>

public class RepositoryViewModel : PropertyChangedBase
{
  public void Delete() {...}    
}
```

## 为什么？

* ReactiveCommand 公开了命令的 `CanExecute` 属性，让程序可以有更多的行为。
* 自动将结果返回到 UI 线程。
* 能够跟踪 in-flight 对象。.



