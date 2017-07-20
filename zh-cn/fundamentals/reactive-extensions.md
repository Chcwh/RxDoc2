Reactive Extensions 的[官方网站](http://reactivex.io/intro.html) （通常称为 ReactiveX ）对该库提供了一个很好的总结：

> ReactiveX 是一个将基于事件的程序异步化的库， 通过使用可观察序列和 LINQ 式查询运算符。

我们将分解这个非常密集的解释（实际上，it could fit in a tweet！），并且逐点讨论在编写用户界面时，该库的功能是否有用。

**注意** 尽管我们尽了最大的努力，我们将无法在本章中覆盖整个 Reactive Extensions。 这个话题是如此巨大，它本身就是一本书。 事实上，有这样的书（甚至是免费的）！ - 看看了解更多部分。

## 可观察序列

Reactive Extensions 最重要的概念是 *数据序列* ，也称为 *流* 。流只是一系列不连续的数值（不管什么类型）。这一系列的值可以是有限的或无限的。有限序列可以正常结束或通过发出错误信号，无限序列只有在发出错误信号或程序退出时才会结束。

事实证明，这个简单的抽象可以用来建模大量不同的系统。这里有些例子：

1. ACME 的股票价格 - 带有时间戳的一系列十进制值。它是有限的 - 它正常结束（当股市关闭时）或错误（当服务器发生连接问题时）。
1. 鼠标位置 - XY 双值的系列，以恒定频率采样。它是无限的 - 程序启动时启动，程序终止时结束。
1. 推文 [@ReactiveXUI](https://twitter.com/ReactiveXUI)  - 一系列字符串。这是一个无限的序列，除非它以 Twitter 停机时引起的错误结束。
1. 按钮点击 - 一系列 `Unit` 值。每次点击按钮时都会显示一个新值。它是无限的 - 仅当程序终止时才结束。

*信息*  `Unit` 类型是一种特殊类型，允许且只允许一个值。等同于着“无（null）”或“无效（void）”。

应该很容易地找到类似的例子。所有这些问题都可以使用 Reactive Extensions 主要类型（即 `IObservable <T>`）轻松建模。

## 用于观察的类型

为了理解 `IObservable<T>` 的作用，请看下面的表格。

|              |单个对象   | 多个对象  |
|:------------:|:-------------:|:-----:|
| 同步  | `T getFoo()` | `IEnumerable<T> getFoos()` |
| 异步 | `Task<T> getFoo()` | `IObservable<T> getFoos()` |

可以看到 `IObservable<T>` 的返回值是异步的，和 `Task<T>` 差不多，除了它能够返回多个元素之外。

可以比较一下 `IObservable<T>` 和 `IEnumerable<T>`。`IEnumerable<T>` **基于拉取** - 就是说必须明确的请求序列中的下一个元素（比如 foreach 循环中，处理完一个元素后，尝试取得下一个）。`IObservable<T>` **基于推送** - 无需询问下一个元素，在下一个元素可用的时候自动送达。

使用 `IObservable<T>` 的语法与 `Task<T>` 和 `IEnumerable<T>` 不同。核心方法叫做 `Subscribe`，其签名如下：

`IDisposable Subscribe(this IObservable<T> self, Action<T> onNext, Action<Exception> onError, Action onCompleted)`

简单用法：

```
IObservable<T> stream = getFoos();
IDisposable disposable = stream.Subscribe(
	element => Console.WriteLine($"New element arrived {element}"),
	error => Console.WriteLine($"Uh oh, an error {error}"),
	() => Console.WriteLine("Stream ended"));
```

正如所见，可以指定新元素抵达时的触发的动作，也可以指定发生错误时的动作，还可以为指定流正常结束的动作。

如果对 `Subscribe` 返回的 `IDisposable` 有所顾虑，可以从流中“取消订阅”。比如：

```
disposable.Dispose();
// 从现在开始不会继续输出 "New element arrived" 
// 即便有新元素可用
```

可以看作 .NET 事件处理程序中的 `-=` 操作。

## 组合性

我们回到我们的单句定义：

> ReactiveX 是一个将基于事件的程序异步化的库， 通过使用可观察序列和 LINQ 式查询运算符。

你已经知道“可观察序列”是什么。 现在，有趣的部分开始了。

`IEnumerable <T>` 接口是整个 LINQ 最好的东西，其使得过滤，转换和组合顺序非常简单。 好消息是 - Reactive Extensions 可以完成和 LINQ 类似的工作。 此外，除了标准的 LINQ 操作，如 `Select`，`Where` 或 `GroupBy` ，Reactive Extensions 提供了一组强大的基于时间的操作。 它们允许（例如）延迟序列元素的到达时间，或者仅当新元素到达太快而无法处理时，才对其进行过滤。

## 异步与基于事件

Reactive Extensions 的定义承诺提供将基于事件的程序异步化的方法。 从 UI 编程的角度来看，库提供一种方便的方式来决定数据传递到哪个同步上下文是很重要的。 很多 UI 框架要求仅从特定的 UI 线程访问 UI 元素。

使用 Reactive Extensions，可以使用 `Scheduler` 类参数化流的并发性。 例如，可以声明所有的数据处理应该在 `TaskPool` 线程上执行，然后（在整个处理之后）直接向 UI 线程传递最终值。 这种机制非常灵活，易于维护，而且 - 最后但并非最不重要 - 易于测试。 由于操作的同步上下文由 `Scheduler` 的抽象层隐藏，可以轻松地模拟它。 这允许轻松地在单元测试中模拟时间流逝。

## 扩展学习

正如我们在开头所说，在一篇文章中描述 Reactive Extensions 库 这样一个巨大的话题是不可能的。事实上，在这个介绍中表面上的都没讲完。

如果想了解更多信息，网上有大量免费资源。首先是 [Introduction to Rx](http://www.introtorx.com)，一本免费的书，以非常方便的方式介绍库的各个方面。 [Reactive Extensions 官方网站](http://reactivex.io/intro.html)上还可以找到另一个很好的学习资料。

如果不能立即阅读关于该主题的整本书，您以查看 [the introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)。这是一个很好的文章，容易跟随，不太浅也不太深。本文是基于 [RxJS]https://github.com/ReactiveX/RxJS)  - 一个JavaScript的风格的库 - 但这不应该是一个问题（一些函数名称可能会有所不同，但概念是完全相同的）。

如果更需要视频教程，请查看 Joe Albahari 的 [Becoming a C# Time Lord](https://channel9.msdn.com/Events/TechEd/Australia/2013/DEV422)。

如果希望看到库使用的一些示例，那么没有比 [Rx 101 samples wiki](http://rxwiki.wikidot.com/101samples) 更好的地方。

我们还鼓励使用 [Rx marbles](http://rxmarbles.com/) - 这是一个互动网站，显示不同的 Rx 操作符的工作方式。

最后，Reactive Extensions 网站上有[很多教程和其他学习资料](http://reactivex.io/tutorials.html)。随意浏览！