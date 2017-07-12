# 绑定

一旦创建了命令，并在视图模型上公开，那么下一步就是在视图上使用了。通常通过 `BindCommand` 方法来完成。该方法有几个重载，可以绑定任何 `ICommand` 源到目标控件。典型用法如下：

```cs
// 在视图中
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl);
```

这里将 `myControl` 控件绑定到视图模型上公开的 `MyCommand` 命令。接下来发生的事情就取决于 [service locator](http://docs.reactiveui.net/en/user-guide/dependency-injection/index.html) 中注册的 `ICreatesCommandBinding` 实例了。无论如何，通常情况下会在命令不可用的时候禁用 `myControl` 。另外，会在 `myControl` 发生某些默认行为的时候执行命令。比如，如果 `myControl` 是按钮，所需的行为就是一个点击（或触摸）。

> **注意** 上面只是单独调用 `BindCommand`。通常情况下会放到 `WhenActivated` 块中：
> 
> ```cs
> this.WhenActivated(
>     d =>
>     {
>         d(this.BindCommand(
>             this.ViewModel,
>             x => x.MyCommand,
>             x => x.myControl));
>     });
> ```
> 
> 更多内容，查阅 [关于 `WhenActivated` 的文档](http://docs.reactiveui.net/en/user-guide/when-activated/index.html) 。

`BindCommand`所演示的这种方式，没有关于将会在那个事件上执行命令的信息。因此将使用默认事件（例如点击或触摸）。另一方面，如果想要将命令的执行与默认事件之外的事件关联，可以使用 `BindCommand` 的重载：

```cs
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl,
    nameof(myControl.SomeEvent));
```

然后，`myControl` 的 `SomeEvent` 将会用于触发命令执行了。

最后，`BindCommand` 的重载允许提供用于执行命令的参数。该参数可以由一个函数、可观察对象甚至是表达式：

```cs
// 使用执行计数作为参数
var count = 0;
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl,
    () => count++);

// 使用可观察对象作为参数
IObservable<int> param = ...;
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl,
    param);

// 使用视图模型上的某个属性作为参数
this.BindCommand(
    this.ViewModel,
    x => x.MyCommand,
    x => x.myControl,
    x => x.SomeProperty);
```