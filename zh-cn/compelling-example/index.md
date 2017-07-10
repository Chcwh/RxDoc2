# Compelling Example

> **信息** 如果你对 Reactive Extensions 和 MVVM 模式不熟悉，应该先了解 [基础章节](../fundamentals/index.md)！

比如有一个文本字段，在用户输入某些东西的时候，联网进行查询。

![](http://i.giphy.com/xTka02wR2HiFOFACoE.gif)

```csharp
public interface ISearchViewModel
{
    ReactiveList<SearchResults> SearchResults { get; }
    string SearchQuery { get; }	 
    ReactiveCommand<List<SearchResults>> Search { get; }
    ISearchService SearchService { get; }
}
```

## 确定什么情况下将会发起网络请求
```csharp
// 在这里以一种 *声明式* 的方式，指定 Search 命令什么时候可以执行。
// 现在命令的 IsEnabled 已经非常高效了，因为只在 UI 需要更新的情况下才会更新。
var canSearch = this.WhenAny(x => x.SearchQuery, x => !String.IsNullOrWhiteSpace(x.Value));
```

## 发起网络请求
```csharp
// ReactiveCommand 自身能够支持后台操作，确保同一时间只执行一次。
// 与此同时，CanExecute 将会自动设置为 false，在执行过程中自动设置 IsExecuting。
Search = ReactiveCommand.CreateAsyncTask(canSearch, async _ => {
    return await searchService.Search(this.SearchQuery);
});
```

## 更新用户界面 
```csharp
// ReactiveCommand 是 IObservable 对象，其值为异步方法的返回值，保证到达 UI 线程。
// 在后台加载搜索结果，并将其填充到 SearchResults。
Search.Subscribe(results => {
    SearchResults.Clear();
    SearchResults.AddRange(results);
});

```

## 处理异常
```csharp
// ThrownExceptions 可以取得由 CreateAsyncTask 创建的 Observable 抛出的异常。   
Search.ThrownExceptions
    .Subscribe(ex => {
        UserError.Throw("Potential Network Connectivity Error", ex);
    });
```

## 设置一个延迟
```csharp
// 在 Search 查询产生变化时，在 1 秒钟内没有新的变化，就会自动调用命令。
this.WhenAnyValue(x => x.SearchQuery)
    .Throttle(TimeSpan.FromSeconds(1), RxApp.MainThreadScheduler)
    .InvokeCommand(this, x => x.Search);
```