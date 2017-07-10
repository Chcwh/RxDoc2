# 函数响应编程

函数响应编程到底是什么？

某种程度上我们知道函数编程是什么。函数没有副作用，没有可变状态。但是真实世界是可变的，编程因此也变得困难。建模用户输入成为了一项噩梦。

*响应* 编程是什么？ 比如说有一个电子表格，有三个单元格 A, B, 和 C。 A 是 B 和 C 的和。每当 B 或 C 改变时， A *自动* 更新自己。这就是响应式编程：变化在系统中自动传播。

函数响应编程只是函数化和响应编程的结合。

Functional reactive programming is the peanut butter and chocolate of programming paradigms.

## Signals

函数响应编程的核心是信号。信号是一个相当抽象的概念，描述信号的作用比描述它是什么更容易。一个信号随着时间自动发送值，一直发送，直到完成或者发生错误并永远终止。信号最终会完成或出错终止，不会同时发生。

信号没有“当前值”，“过去值”，“未来值”的概念，它们非常简单。他们只是 *随着时间发送值* 。当它们被订阅或在绑定中用到时，才变得有用。重要的是，要懂得信号可以 *被转换*。

如果有一个发送数字的信号，我希望它发送字符串，就可以使用 *map* 轻松的 *转换* 信号。map 是一个 *运算符* ，其需要一个信号作为输入，在值上面做一些操作，生成一个新的信号，发送转换后的值。

信号可以 *被订阅* 。比如说当一个按钮被按下时发出的信号，可以订阅这个信号，然后做一些操作。信号也可以 *被绑定* 到属性。比如说发送手势识别器位置的信号可以映射到颜色，并绑定到视图的背景色。然后，当用户改变触摸的位置时，视图改变颜色。

订阅和绑定的例子，都表明了在应用程序初始化的时候，告诉代码 *做什么* ，而不是 *怎么做* 。这就是函数响应编程的价值所在。

Another core concept in functional reactive programming is that of *derived* state. State is OK in ReactiveUI, as long as it's *bound* to a signal. This derived state means that you never explicitly set the value of a bound property, but rather rely on signal transformations to derive that state for you.

We recommend reading [this academic paper](https://raw.githubusercontent.com/papers-we-love/papers-we-love/master/design/out-of-the-tar-pit.pdf) and/or playing with the [Elm language](http://elm-lang.org) if you need a deeper explanation.

