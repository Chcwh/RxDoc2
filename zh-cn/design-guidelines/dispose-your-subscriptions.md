# 销毁订阅

http://www.introtorx.com/content/v1.0.10621.0/03_LifetimeManagement.html

```
this.WhenActivated(
    disposables =>
    {
        this.WhenAnyValue(...)
            .DisposeWith(disposables);
    });
```

还有 https://docs.reactiveui.net/en/user-guide/when-activated/


注意所有的订阅都需要销毁。就和事件一样。如果一个组件公开一个事件，并自己订阅，是不需要解除订阅的。因为订阅表现为组件拥有自己的引用。Rx 也是一样。如果一个视图模型通过诸如 `WhenAnyValue` 的方式与自己的属性关联，是不需要清理的，因为这表现为视图模型拥有自己的引用。
