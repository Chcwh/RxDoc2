# Example

# Reactive UI 101

现在创建一个简单的应用程序来演示一些 ReactiveUI 功能，不会引入太多的内部细节。


应用程序的完整代码在本章末尾，将在相关代码段中显示。

在 Visual Studio 中创建一个新的 WPF 应用程序（.Net 4.5或更高版本）

视图 `MainWindow` 已经创建，所以将继续创建视图模型。

**添加引用**
```csharp
using System.Web;
```

**添加 NuGet 包**
```
Install-Package ReactiveUI
```

**添加字段**
```csharp
public AppViewModel ViewModel { get; private set;} 
```

**在 MainWindow 构造函数中创建一个值，并绑定到 DataContext**

```csharp
public MainWindow()
{
    ViewModel = new AppViewModel();
    InitializeComponent();
    DataContext = ViewModel;
}  
```

**创建 "AppViewModel" 类**
```csharp
// AppViewModel 里面包含了应用程序的交互逻辑
// （由于逻辑简单的缘故，整个程序的逻辑都可以放在一个类里面）
public class AppViewModel : ReactiveObject
{
	// 在 ReactiveUI 中，这是定义可以通知观察者属性改变的可读写属性的语法。
    string _SearchTerm;
    public string SearchTerm
    {
        get { return _SearchTerm; }
        set { this.RaiseAndSetIfChanged(ref _SearchTerm, value); }
    }

    public ReactiveCommand<string, List<FlickrPhoto>> ExecuteSearch { get; protected set; }
	 
    /* ObservableAsPropertyHelper
     * 
     * 在 ReactiveUI 中，可以将可观察对象“引导”到一个属性上。
     * 在可观察对象有一个新值的时候，将会通知 ReactiveObject 该属性发生了改变。
     * 
     * 要做到这一点，可以使用 ObservableAsPropertyHelper 类。该类可以订阅一个
     * 可观察对象，并存储最新值的副本。并在属性发生该改变时执行某些动作，通常
     * 是调用 ReactiveObject 的 RaisePropertyChanged。
     * 
     * 
     */	 
    ObservableAsPropertyHelper<List<FlickrPhoto>> _SearchResults;
    public List<FlickrPhoto> SearchResults => _SearchResults.Value;

	// 创建一个指示是否正在执行搜索的属性（比如显示 “Spinner” 控件，让用户知道程序忙）。
	// 可以将这个属性作为某个可观察对象的结果（比如其值由其它属性驱动）
    ObservableAsPropertyHelper<Visibility> _SpinnerVisibility;
    public Visibility SpinnerVisibility => _SpinnerVisibility.Value;

    public AppViewModel()
    {
        ExecuteSearch = ReactiveCommand.CreateFromTask<string, List<FlickrPhoto>>(
            searchTerm => GetSearchResultsFromFlickr(searchTerm)
        );

		
        /* 以声明的方式创建 UI
		/*
		 * 视图模型中的属性以各种方式相互关联。在其他框架中，要简洁的描述这些关系很困难。
		 * 实现“在搜索进行时显示 UI spinner ”的代码都可能需要数个事件处理程序。
		 *
		 * 但是，使用 RxUI ，可以用浅显易懂的方式描述属性之间的关系。
		 * 以用户完成的顺序，描述用户的工作流。
		 */
		
		// 将一个属性变成可观察对象，该可观察对象在属性发生改变时，发出一个值。
		
		// 使用 Throttle 操作符来过滤过于频繁的变化，因为不想为每次键入进行搜索。
		// 然后获取改变的值，并过滤掉相同的变化，以及空值。
		
		// 最后，使用 RxUI 的 InvokeCommand 操作符，调用 ExecuteSearch 命令的的 Execute 方法
		
        this.WhenAnyValue(x => x.SearchTerm)
            .Throttle(TimeSpan.FromMilliseconds(800), RxApp.MainThreadScheduler)
            .Select(x => x?.Trim())
            .DistinctUntilChanged()
            .Where(x => !String.IsNullOrWhiteSpace(x))
            .InvokeCommand(ExecuteSearch);
		
		// 用文字怎么描述什么时候显示 spinner ：“ spinner 的可见性取决于是否正在搜索 ”
		// 使用 RxUI ，可以在代码中也这么写
		
		// ExecuteSearch 有一个叫做 IsExecuting 的 IObservable<bool> 。
		// 其在每次命令改变执行状态的时候都会触发。
		// 可以使用个 Select() 将其转换为 Visibility。
		// 然后使用 ToProperty 操作符，创建一个 ObservableAsPropertyHelper 对象。

        _SpinnerVisibility = ExecuteSearch.IsExecuting
            .Select(x => x ? Visibility.Visible : Visibility.Collapsed)                
            .ToProperty(this, x => x.SpinnerVisibility, Visibility.Hidden);
        
		// 订阅 ReactiveCommand 的 ThrownExceptions 属性，可以获取执行过程中抛出的异常。
		// 更多细节，查看“错误处理”章节。		
        ExecuteSearch.ThrownExceptions.Subscribe(ex => {/* Handle errors here */});

		// 这里，将描述命令调用时执行的内容。这里是执行 GetSearchResultsFromFlickr。
		
		// 重点是返回值 ———— 一个可观察对象。这里是 FlickrPhoto 列表的流：
		// 每次执行，都会得到一个新的列表，并立即赋值给 SearchResults 属性，
		// 并自动出发 INotifyPropertyChanged 。
        _SearchResults = ExecuteSearch.ToProperty(this, x => x.SearchResults, new List<FlickrPhoto>());
    }

    public static async Task<List<FlickrPhoto>> GetSearchResultsFromFlickr(string searchTerm)
    {
        var doc = await Task.Run(() => XDocument.Load(String.Format(CultureInfo.InvariantCulture,
            "http://api.flickr.com/services/feeds/photos_public.gne?tags={0}&format=rss_200",
            HttpUtility.UrlEncode(searchTerm))));

        if (doc.Root == null)
            return null;

        var titles = doc.Root.Descendants("{http://search.yahoo.com/mrss/}title")
            .Select(x => x.Value);

        var tagRegex = new Regex("<[^>]+>", RegexOptions.IgnoreCase);
        var descriptions = doc.Root.Descendants("{http://search.yahoo.com/mrss/}description")
            .Select(x => tagRegex.Replace(HttpUtility.HtmlDecode(x.Value), ""));

        var items = titles.Zip(descriptions,
            (t, d) => new FlickrPhoto { Title = t, Description = d }).ToArray();

        var urls = doc.Root.Descendants("{http://search.yahoo.com/mrss/}thumbnail")
            .Select(x => x.Attributes("url").First().Value);

        var ret = items.Zip(urls, (item, url) => { item.Url = url; return item; }).ToList();
        return ret;
    }
}
```

The goal of the syntax of ReactiveUI for read-write properties is to notify Observers that a property has changed. 
Otherwise we would not be able to know when it was changed. 

The ExecuteSearch is basically an asynchronous task, executing in the background.
  
In cases when we don't need to provide for two-way binding between the View and the ViewModel, we can use
one of many ReactiveUI Helpers, to notify Observers of a changing read-only value in the ViewModel. We use the
ObservableAsPropertyHelper twice, once to turn a generic List<T> into an observable read-only collection,
and then to change the visibility of an indicator to show that a request is currently executing.

This also works in the opposite direction, when we take the `SearchTerm` property and turn it into an observable. This means that we are notified every time a change occurs in the UI. Using Reactive Extensions, we then throttle those events,
and ensure that the search occurs no sooner than 800ms after the last keystroke. And if at that point the user did not change the
last value, or if the search term is blank, we ignore the event completely.

Using the `IsExecuting` observable of `ReactiveCommand`, we derive another 
observable to change the visibility of the "processing indicator".

The `GetSearchResultsFromFlickr` method gets invoked every time there is a 
throttled change in the UI, so let's define what should happen when a user executes a new search.

Create a simple model class to hold the Flickr results - since we never update the properties once we've created the object, we don't have to use a ReactiveObject.

ReactiveUI 读写属性的语法的目的是通知观察者们属性已更改。否则在改变的时候就不知道了。

ExecuteSearch 基本上是一个异步的任务，在后台执行。

在我们不需要在视图和视图模型之间提供双向绑定的情况下，我们可以使用许多 ReactiveUI Helper 中的一个来通知观察者们 视图模型中某个只读值的更改。这里使用了 ObservableAsPropertyHelper 两次，一次将通用列表转换为可观察的只读集合，另一次是更改 spinner 的可见性，以显示当前正在执行请求。

当采用 `SearchTerm` 属性并将其转换成可观察值时，也可以在相反的方向工作。这意味着每当 UI 中发生更改时都会收到通知。使用Reactive Extensions，然后过滤这些事件，并确保搜索发生在最后一次键入的 800ms 之后。而且如果用户没有更改最后一个值，或者如果搜索项是空白的，那么完全忽略该事件。

使用 `ReactiveCommand` 的 `IsExecuting` 可观察对象，我们导驱动另一个可观察对象来更改 “spinner” 的可见性。

每次在 UI 中发生已过滤的更改时，都会调用 `GetSearchResultsFromFlickr` 方法，因此可以定义当用户执行新的搜索时的执行逻辑。

创建一个简单的模型类来保存 Flickr 结果 - 因为一旦创建了对象，就不会更新属性，所以不必继承 ReactiveObject。

```csharp
namespace FlickrBrowser
{
    public class FlickrPhoto
    {
        public string Title { get; set; }
        public string Description { get; set; }
        public string Url { get; set; }
    }
}
```

**完成视图模型**

为视图模型创建视图，如下：

```xml
<Window x:Class="FlickrBrowser.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Name="Window" Height="350" Width="525">
    <Window.Resources>
        <DataTemplate x:Key="PhotoDataTemplate">
            <Grid MaxHeight="100">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto" />
                    <ColumnDefinition Width="*" />
                </Grid.ColumnDefinitions>

                <Image Source="{Binding Url, IsAsync=True}" Margin="6" MaxWidth="128"
                       HorizontalAlignment="Center" VerticalAlignment="Center" />

                <StackPanel Grid.Column="1" Margin="6">
                    <TextBlock FontSize="14" FontWeight="Bold" Text="{Binding Title}" />
                    <TextBlock FontStyle="Italic" Text="{Binding Description}" 
                               TextWrapping="WrapWithOverflow" Margin="6" />
                </StackPanel>
            </Grid>
        </DataTemplate>
    </Window.Resources>

    <Grid Margin="12">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" />
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="Auto" />
        </Grid.ColumnDefinitions>

        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <TextBlock FontSize="16" FontWeight="Bold" VerticalAlignment="Center">Search For:</TextBlock>
        <TextBox Grid.Column="1" Margin="6,0,0,0" Text="{Binding SearchTerm, UpdateSourceTrigger=PropertyChanged}"/>
        <TextBlock Grid.Column="2" Margin="6,0,0,0" FontSize="16" FontWeight="Bold" Text="..." Visibility="{Binding SpinnerVisibility}" />

        <ListBox Grid.ColumnSpan="3" Grid.Row="1" Margin="0,6,0,0" 
                 ScrollViewer.HorizontalScrollBarVisibility="Disabled"
                 ItemsSource="{Binding SearchResults}" ItemTemplate="{DynamicResource PhotoDataTemplate}"  />
    </Grid>
</Window>
```   
