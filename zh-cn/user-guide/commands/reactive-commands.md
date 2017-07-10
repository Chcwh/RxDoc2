# Reactive Commands

响应命令还封装了响应用户操作的执行逻辑，但是通过一个响应式API来完成。 例如，不用查询命令的可执行性，而是使用 `IObservable<bool>` 的 `CanExecute` 属性。 类似地，`IsExecuting` 属性（类型也为 `IObservable <bool>`）告诉你该命令当前是否正在执行。 例如，这可能是一个启用活动动画的触发器。 当执行该命令时，将获得一个 `IObservable <TResult>`。 `TResult` 是执行命令的“结果”（通常是 `Unit`，如果命令不需要返回任何重要的东西）。

执行命令返回的结果是可观察对象，而不是结果本身，清楚地表明，响应式命令本质上是异步的。您可以启动CPU或基于的I/O命令，而不会阻塞您的UI。一旦完成，结果将通过可观察对象返回，用户界面可以相应地进行响应。当命令正在执行时，它不可用（即 `CanExecute` 将为 `false`）。绑定到该命令的任何UI元素将在命令执行时自动禁用自身。

响应命令本身是可观察的。无论命令何时执行完成，结果将通过命令本身进行tick。

响应命令的一个重要事实是，它们保证在给定的调度器上传递事件，但是它们不会将用户提供的管道（包括执行逻辑）组织到该调度程序。因此，从正确线程的进行命令交互是调用者的责任。 But any event you receive from a reactive command is guaranteed to be surfaced via the provided scheduler.

> **提示** 所有响应命令都实现了 `ICommand`，但是是显式的。 这是希望使用响应式 API，而不是显式实现的 `ICommand`。然而，它也允许响应命令无缝集成到与 `ICommand` 有关的框架中



