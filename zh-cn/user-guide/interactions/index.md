# 交互
有时您可能会发现自己编写的视图模型代码需要用户确认某些内容。 例如，检查是否可以删除文件，或询问如何处理发生的错误。

从视图模型中直接弹出一个消息框可能很方便。 但这是不正确的。 这不仅将您的视图模型与特定的 UI 技术联系起来，还使测试变得困难（甚至是不可能的）。

相反，需要的是一种挂起视图模型执行的手段，直到用户提供某些数据。 ReactiveUI 的交互机制可以做到这一点。

## API Overview

互动架构的基础是 `Interaction <TInput，TOutput>` 类。这个类提供了交互的协作组件之间的中介。它负责协调和分配与处理程序的交互。

交互接受输入并产生输出。输入是在处理交互时可以使用的视图。输出是视图模型从交互中返回的东西。例如，想象一个需要询问用户是否可以删除文件的视图模型。为此，它可以传递文件的名称作为输入，并返回一个布尔值作为输出，指示文件是否可以被删除。

交互的输入和输出类型完全由您控制。它们 `Interaction <TInput，TOutput>` 的泛型类型参数 `TInput` 和 `TOutput` 推定。因此，并不严格限制的输入，也不管输出什么。
 
> **注意** 有时可能不会关心输入类型。在这种情况下，可以使用 `Unit` 。也可以使用 `Unit` 作为输出类型，尽管这意味着您的视图模型没有使用交互作出决定。相反，它仅仅是通知即将要发生的事情。

交互处理程序接收到 `InteractionContext <TInput，TOutput>`。交互上下文通过 `Input` 属性暴露交互的输入。另外，它为处理程序提供了通过调用 `SetOutput` 方法提供交互输出的方法。

交互组件的常见配置：

* **视图模型**: 想知道问题的答案，比如“是否可以删除文件？”
* **视图**: 询问问题，并在交互期间提供答案

虽然这种配置最常见，但绝不是必需的。 例如，可以在没有用户干预的情况下将视图自己回答问题。 或者两个组件都是视图模型。 ReactiveUI 提供的交互架构不会对协作组件施加任何限制。

假设为常见配置，视图模型将创建并公开一个 `Interaction <TInput，TOutput>` 的实例。 相应的视图将通过调用其中一个 `RegisterHandler` 方法来注册一个处理这个交互的处理程序。 为了引发交互，视图模型将传入一个 `TInput` 的实例到 `Handle` 方法。 它将异步地接收到类型为 `TOutput` 的结果。

## 例子

```cs
public class ViewModel : ReactiveObject
{
    private readonly Interaction<string, bool> confirm;
    
    public ViewModel()
    {
        this.confirm = new Interaction<string, bool>();
    }
    
    public Interaction<string, bool> Confirm => this.confirm;
    
    public async Task DeleteFileAsync()
    {
        var fileName = ...;
        
        // 如果没有东西处理交互，那么将会抛出异常
        var delete = await this.confirm.Handle(fileName);
        
        if (delete)
        {
            // delete the file
        }
    }
}

public class View
{
    public View()
    {
        this.WhenActivated(
            d =>
            {
                d(this
                    .ViewModel
                    .Confirm
                    .RegisterHandler(
                        async interaction =>
                        {
                            var deleteIt = await this.DisplayAlert(
                                "Confirm Delete",
                                $"Are you sure you want to delete '{interaction.Input}'?",
                                "YES",
                                "NO");
                                
                            interaction.SetOutput(deleteIt);
                        }));
            });
    }
}
```

也可以创建在多个组件间共享的 `Interaction<TInput, TOutput>`。一个常见的例子是错误恢复。 许多组件可能希望引发错误，但是我们可能只需要一个通用的处理程序。 举个例子：

```cs
public enum ErrorRecoveryOption
{
    Retry,
    Abort
}

public static class Interactions
{
    public static readonly Interaction<Exception, ErrorRecoveryOption> Errors = new Interaction<Exception, ErrorRecoveryOption>();
}

public class SomeViewModel : ReactiveObject
{
    public async Task SomeMethodAsync()
    {
        while (true)
        {
            Exception failure = null;
            
            try
            {
                DoSomethingThatMightFail();
            }
            catch (Exception ex)
            {
                failure = ex;
            }
            
            if (failure == null)
            {
                break;
            }
            
            // this will throw if nothing handles the interaction
            var recovery = await Interactions.Errors.Handle(failure);
            
            if (recovery == ErrorRecoveryOption.Abort)
            {
                break;
            }
        }
    }
}

public class RootView
{
    public RootView()
    {
        Interactions.Errors.RegisterHandler(
            async interaction =>
            {
                var action = await this.DisplayAlert(
                    "Error",
                    "Something bad has happened. What do you want to do?",
                    "RETRY",
                    "ABORT");

                interaction.SetOutput(action ? ErrorRecoveryOption.Retry : ErrorRecoveryOption.Abort);
            });
    }
}
```

> **注意** 为了清楚起见，这里的示例代码混合了 TPL 和 Rx 代码。 生产代码通常会只会使用一个。

> **警告** `Handle` 返回的可观察对象是冷的。必须对他进行订阅，以调用处理程序。

## 成立程序优先级

`Interaction<TInput，TOutput>` 实现一个处理程序链。可以注册任何数量的处理程序，后注册的比先注册的更为优先。当使用` Handle` 方法启动交互时，每个处理程序都 _有机会_处理该交互（即设置一个输出）。处理程序没有义务实际处理交互。如果处理程序选择不设置输出，则链中的下一个处理程序将被调用。

> **注意** `Interaction<TInput，TOutput>` 类被设计为可扩展的。子类可以改变 `Handle` 的行为，使得它不会表现出上述行为。例如，您可以编写一个仅执行列表中第一个处理程序的实现。

这个优先级链可以定义一个默认处理程序，然后暂时覆盖该处理程序。例如，根级别处理程序可以提供默认错误恢复行为。但是，应用程序中的具体视图可能会知道如何在不提示用户的情况下从某个错误中恢复。它可以在激活时注册一个处理程序，然后在停用时处理该注册。显然，这种方法需要共享交互实例。

## 未处理的交互

如果某个交互没有定义处理程序，或者所有的处理程序都没有设置输出，那么交互就没有被处理。在这种情况下，`Handle` 的调用将会引发 `UnhandledInteractionException` 异常。该异常包含了 `Interaction` 和 `Input` 属性，因此可以检查失败交互的详情。

## 测试

通过为交互注册处理程序，可以轻易的测试交互逻辑：

```cs
[Fact]
public async Task interaction_test()
{
    var fixture = new ViewModel();
    fixture
        .Confirm
        .RegisterHandler(interaction => interaction.SetOutput(true));
        
    await fixture.DeleteFileAsync();
    
    Assert.True(/* file was deleted */);
}
```

如果测试与某个共享的交互挂钩，那么在测试返回前，需要销毁注册：

```cs
[Fact]
public async Task interaction_test()
{
    var fixture = new SomeViewModel();
    
    using (Interactions.Error.RegisterHandler(interaction => interaction.SetOutput(ErrorRecoveryOption.Abort)))
    {
        fixture.SomeMethodAsync();
        
        // 断言在这里中止
    }
}
```