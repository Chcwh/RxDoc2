# Functional Reactive Programming

Functional. Reactive. Programming. What on Earth is that?

Well, we know to some degree what functional programming is. Functions don't have side-effects; there's no mutable state. It's a difficult way to program since the real world is mutable. Modeling things like user input becomes a nightmare.

*Reactive* programming? What's that? Well, the best way to describe reactive programming is to think of a spreadsheet. Imagine three cells, A, B, and C. A is defined as the sum of B and C. Whenever B or C changes, A *reacts* to update itself. That's reactive programming: changes propagate throughout a system automatically.

Functional reactive programming is just a combination of functional and reactive paradigms. We model user input as a function that changes over time, abstracting away the idea of mutable state.

Functional reactive programming is the peanut butter and chocolate of programming paradigms.

## Signals

At the core of functional reactive programming is a signal. A signal is a rather abstract concept, and it's easier to describe what a signal does rather than what it is. A signal simply sends values over time. It sends values until it either completes, or errors out, at which point it stops sending values forever. A signal either completes or errors out, but not both.

Signals have no sense of a "current value" or "past values" or "future values", they are very simple. They just *send values over time*. Signals become useful when they're subscribed to or used with bindings, both of which we'll cover shortly. For now, it's important to understand that a signal can be *transformed*.

If I have a signal that sends numbers, and I want it to send strings instead, I can easily *transform* that signal with a *map*. A map is an *operator* that takes in a signal as a receiver, performs some operation on the values sent over that signal, and generates a new signal with the new, mapped values.

A signal can be *subscribed to* in order to perform side-effects. A good example would be the signal emitted by a button when it's pressed; we can subscribe to that signal in order to perform side effects. A signal can also be *bound* to a property. For example, a signal that sends the location of a gesture recognizer can be mapped to a color and bound to the background color of a view. Then, when the user changes their tap position, the view changes color.

Both of these examples, of subscription and binding, demonstrate that we can create a set up at the beginning of an application run-time and tell the code *what to do* without telling it *how to do it*. This is the value of functional reactive programming.

Another core concept in functional reactive programming is that of *derived* state. State is OK in ReactiveUI, as long as it's *bound* to a signal. This derived state means that you never explicitly set the value of a bound property, but rather rely on signal transformations to derive that state for you.

We recommend reading [this academic paper](https://raw.githubusercontent.com/papers-we-love/papers-we-love/master/design/out-of-the-tar-pit.pdf) and/or playing with the [Elm language](http://elm-lang.org) if you need a deeper explanation.

