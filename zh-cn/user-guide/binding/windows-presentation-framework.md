# Windows Presentation Framework

在 WPF 中很重要的一点是使用 `WhenActivated` 方法，以避免内存泄露  -> https://codereview.stackexchange.com/questions/74642/a-viewmodel-using-reactiveui-6-that-loads-and-sends-data


    如果不使用 `WhenActivated` ，XAML DependencyProperty 会导致内存泄露。
	有一些规则，但第一条就是：如果在 `this` 以外的地方使用了 `WhenAny` 应该将其放到 `WhenActivated` 中

    this.WhenActivated(d =>
    {
       d(ViewModel.WhenAnyValue(x => x.Something).Subscribe(...));
    });

# Binding via Codebehind

# Static Binding via {x:Bind}

# Reflection based binding via {Binding}


# Chatlog

    是否还不能在 ItemTemplate 内部使用 RXUI 绑定？
    
    paulcbetts [5:35 AM] 
    是的
    
    ionoy [5:35 AM] 
    该死
    
    paulcbetts [5:35 AM] 
    Well, inside an implicit DataTemplate
    
    paulcbetts [5:36 AM]
    You have to create a UserControl
    
    paulcbetts [5:36 AM]
    The good news is, if you use `OneWayBind` to set the ItemsSource, we'll set up a DataTemplate for you
    
    ionoy [5:37 AM] 
    can you elaborate? if I have an Items collection and one way bind it to my list, how should I name this UserControl then? (for you to automatically set it as DataTemplate) (edited)
    
    paulcbetts [5:38 AM] 
    You can name it whatever you want, as long as:
    1. The UserControl implements `IViewFor<TheViewModelYourePuttingIntoThatList>`
    2. You register the View: `Locator.CurrentMutable.Register(() => new MyUserControl, typeof(IViewFor< TheViewModelYourePuttingIntoThatList>));`
    
    ionoy [5:38 AM] 
    oh, awesome
    
    paulcbetts [5:39 AM] 
    The end result is, in ReactiveUI you end up writing a bit more UserControls, but your XAML is cleaner
    
    ionoy [5:39 AM] 
    this is actually a very good solution
    
    paulcbetts [5:39 AM] 
    Because your ListBox can now be as simple as `<ListBox x:Name="MyList" />`
    
    ionoy [5:39 AM] 
    yep
    
    ionoy [5:49 AM] 
    hm, one downside is that my design time data isn't displayed nicely anymore...
    
    paulcbetts [5:59 AM] 
    Design time data is the best part of ReactiveUI
    
    paulcbetts [5:59 AM]
    So, the way you do design time data, is that you just *set the property* to whatever you want
    
    paulcbetts [5:59 AM]
    And at runtime, the RxUI binding will override it
    
    paulcbetts [6:00 AM]
    It works with ItemsControls too
    
    paulcbetts [6:01 AM]
    ```<ListBox x:Name="Foo">
    <ListBox.Items>   <!-- At runtime, ItemsSource trumps Items -->
    <MyControl />
    <MyControl />
    <MyControl />
    </ListBox.Items>
    </ListBox>
    
    
    moswald [6:14 AM] 
    the only downside is that if you are setting your `ItemsSource` from within a call to `WhenActivated` , your design-time data ends up showing up for a frame or two
    
    moswald [6:14 AM]
    the solution is to manually clear your design-time data at the bottom of your constructor in those cases
    
    ionoy [6:56 AM] 
    @moswald: I set design time data via '{Binding Source={d:DesignInstance Type=models:DesignTimeData, IsDesignTimeCreatable=True}}', so it doesn't show when app is started



Implement `IViewFor<T>` by hand and ensure that ViewModel is a DependencyProperty.  
Also, always dispose bindings view `WhenActivated`, or else the bindings leak memory.
  
The goal in this example is to two-way bind the `TheText` property of the
ViewModel to the TextBox and one-way bind the `TheText` property to the TextBlock, 
so the TextBlock updates when the user types text into the TextBox.
  
```csharp
public class TheViewModel : ReactiveObject
{
    private string theText;
    
    public string TheText
    {
        get { return this.theText; }
        set { this.RaiseAndSetIfChanged(ref this.theText, value); }
    }
}
```

```xml
<Window /* snip */>
  <StackPanel>
    <TextBox x:Name="TheTextBox" />
    <TextBlock x:Name="TheTextBlock" />
  </StackPanel>
</Window>
```

```csharp
public partial class TheView : IViewFor<TheViewModel>
{
    public TheView()
    {
        InitializeComponent();
        
        ViewModel = new TheViewModel();
        
        // Setup the bindings
        // Note: We have to use WhenActivated here, since we need to dispose the
        // bindings on XAML-based platforms, or else the bindings leak memory.
        this.WhenActivated(d =>
        {
            d(this.Bind(this.ViewModel, x => x.TheText, x => x.TheTextBox.Text));
            d(this.OneWayBind(this.ViewModel, x => x.TheText, x => x.TheTextBlock.Text));
        });
    }

    object IViewFor.ViewModel
    {
        get { return ViewModel; }
        set { ViewModel = (TheViewModel)value; }
    }

    public TheViewModel ViewModel
    {
        get { return (TheViewModel)GetValue(ViewModelProperty); }
        set { SetValue(ViewModelProperty, value); }
    }

    public static readonly DependencyProperty ViewModelProperty =
        DependencyProperty.Register("ViewModel", typeof(TheViewModel), typeof(TheView));
}
```
