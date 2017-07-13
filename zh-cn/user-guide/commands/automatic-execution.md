# 自动执行

不要在视图模型中进行，而是在视图中！

详见 -> https://codereview.stackexchange.com/questions/74642/a-viewmodel-using-reactiveui-6-that-loads-and-sends-data

唯一不同的是，不在视图模型的构造函数中直接调用 LoadItems.ExecuteAsyncTask 。在构造函数中调用意味着视图模型难以测试，因为总是模拟出了 LoadItems ，即使要测试的东西与之无关。

作为替代，应该在视图的构造函数中调用，比如：

    this.WhenAnyValue(x => x.ViewModel.LoadItems)
        .SelectMany(x => x.ExecuteAsync())
        .Subscribe();

这就是说在任何取得新视图模型的情况下，会执行 LoadItems，这是我们所希望在程序运行时发生的，而不是在单元测试的时候。        
