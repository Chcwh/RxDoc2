# 事件

 **reactiveui-*-events** 与 ReactiveUI 关系不大。可以单独使用。他们只是为每个 .NET 事件生成了相应的 `IObservable<TEventArgs>` 包装器，这样就不用再自己写 `FromEvent`/`FromEventPattern` 了。

事件包由 [a Moustache template](https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI.Events/Events.mustache)  和 [console app + ruby script](https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI.Events/generate_events.rb) 生成。