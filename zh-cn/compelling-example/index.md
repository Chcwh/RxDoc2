# Compelling Example

> **Info** If you are not familiar with Reactive Extensions and MVVM pattern, you might want to check out our [Fundamentals section](../fundamentals/index.md) first!

Let’s say you have a text field, and whenever the user types something into it, you want to make a network request which searches for that query.

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

## Define under what conditions a network request will be made
```csharp
// Here we're describing here, in a *declarative way*, the conditions in
// which the Search command is enabled.  Now our Command IsEnabled is
// perfectly efficient, because we're only updating the UI in the scenario
// when it should change.
var canSearch = this.WhenAny(x => x.SearchQuery, x => !String.IsNullOrWhiteSpace(x.Value));
```

## Make the network connection
```csharp
// ReactiveCommand has built-in support for background operations and
// guarantees that this block will only run exactly once at a time, and
// that the CanExecute will auto-disable and that property IsExecuting will
// be set according whilst it is running.
Search = ReactiveCommand.CreateAsyncTask(canSearch, async _ => {
    return await searchService.Search(this.SearchQuery);
});
```

## Update the user interface 
```csharp
// ReactiveCommands are themselves IObservables, whose value are the results
// from the async method, guaranteed to arrive on the UI thread. We're going
// to take the list of search results that the background operation loaded, 
// and them into our SearchResults.
Search.Subscribe(results => {
    SearchResults.Clear();
    SearchResults.AddRange(results);
});

```

## Handling failures
```csharp
// ThrownExceptions is any exception thrown from the CreateAsyncTask piped
// to this Observable. Subscribing to this allows you to handle errors on
// the UI thread. 
Search.ThrownExceptions
    .Subscribe(ex => {
        UserError.Throw("Potential Network Connectivity Error", ex);
    });
```

## Throttling network requests and automatic search execution behaviour
```csharp
// Whenever the Search query changes, we're going to wait for one second
// of "dead airtime", then automatically invoke the subscribe command.
this.WhenAnyValue(x => x.SearchQuery)
    .Throttle(TimeSpan.FromSeconds(1), RxApp.MainThreadScheduler)
    .InvokeCommand(this, x => x.Search);
```