The [official website](http://reactivex.io/intro.html) of Reactive Extensions (often called ReactiveX) provides a well worded summary of the library:

> ReactiveX is library for composing asynchronous and event-based programs by using observable sequences and LINQ-style query operators.

We will break down this very dense explanation (actually, it could fit in a tweet!), and discuss point by point how the features of the library are useful while programming user interfaces.

> **Note** Despite our best efforts, we will not be able to cover the whole Reactive Extensions in this chapter. This topic is so huge it deserves a book on its own. And in fact, there are such books (even free ones!) - check out the Learn more section.

## Observable sequences
The most important concept of Reactive Extensions is a *sequence of data*, also know as a *stream*. The stream is simply a discrete series of values (of whatever type). This series of values can be either finite or infinite. The finite sequences can end normally or by signalling an error, and the infinite sequences end only if they signal an error or when your program exits.

As it turns out, this simple abstraction can be used to model a huge amount of different systems. Here are some examples:

1. Stock prices of ACME - a series of decimal values with timestamps. It is finite - it ends normally (when stock market closes) or with error (when there is a connection issue with the server).
1. Mouse position - series of XY double values, sampled with constant frequency. It is infinite - starts when the program starts, ends when the program terminates.
1. Tweets by [@ReactiveXUI](https://twitter.com/ReactiveXUI) - a series of strings. It is a sequence that is infinite, unless it ends with an error caused by Twitter downtime.
1. Button clicks - a series of `Unit` values. A new value appears each time the button is clicked. It is infinite - ends only when program terminates.

> **Info** Unit type is a special type that allows one, and only one, value. Think of it as meaning "nothing" or "void".

You should easily come out with your own examples of streams similar to the listed ones. All of these problems can be easily modelled using Reactive Extensions primary type, which is `IObservable<T>`.

## The type to observe them all
To understand  the role of `IObservable<T>` let's analyze the diagram below displaying several options of a return type from a function. Available options are categorized based on two factors: the number of elements returned from a function and whether function returns synchronously (is blocking) or asynchronously (is not blocking).

|              |single item   | multiple items  |
|:------------:|:-------------:|:-----:|
| synchronous  | `T getFoo()` | `IEnumerable<T> getFoos()` |
| asynchronous | `Task<T> getFoo()` | `IObservable<T> getFoos()` |

You can see that `IObservable<T>` returns asynchronously, which makes it similar to `Task<T>`, except it is able to return more than a single element.

You can also compare `IObservable<T>` to `IEnumerable<T>`.  `IEnumerable<T>` is **pull-based** - that means that you have to explicitly ask it to give you a next element in the sequence (e.g. in a foreach loop, after processing one element, you try to get the next one). `IObservable<T>` is **push-based** - you do not have to ask for a next element in a sequence, it is delivered to the client whenever a new sequence element is available.

The syntax for consuming  `IObservable<T>` differs slightly from using both `Task<T>` and `IEnumerable<T>`. The core method is called `Subscribe`, and has following signature:

`IDisposable Subscribe(this IObservable<T> self, Action<T> onNext, Action<Exception> onError, Action onCompleted)`

And is used simply:

```
IObservable<T> stream = getFoos();
IDisposable disposable = stream.Subscribe(
	element => Console.WriteLine($"New element arrived {element}"),
	error => Console.WriteLine($"Uh oh, an error {error}"),
	() => Console.WriteLine("Stream ended"));
```

As you can see, you are able to specify actions which will be triggered whenever a new element arrives ("New element arrived"). We can also specify what should happen in the event of an error ("Uh oh"), as well as stream ending normally ("Stream ended").

If you're wondering about the returned `IDisposable` from `Subscribe`, it is how you "unsubscribe" from the stream. Like this:

```
disposable.Dispose();
// no more "New element arrived" printed from now on
// even if there is a new element available
```

You can think of it as an equivalent of `-=` operator for standard .Net event handlers.

## Composability
Let's go back to our single-sentence definition:

> ReactiveX is library for composing asynchronous and event-based programs by using observable sequences and LINQ-style query operators.

You already know what "observable sequences" are all about. Now, the fun part begins. 

You should agree that the best thing about `IEnumerable<T>` interface is the whole LINQ thing, making filtering, transforming and combining sequences very easy. Good news - Reactive Extensions allow you to do all the things you know from LINQ! Moreover, apart from standard LINQ operations like `Select`, `Where` or `GroupBy`, Reactive Extensions provides you a set of powerful time based operations. They let you (for example) delay the arrival time of elements of a sequence, or filter them only when the new elements arrive too fast to be processed.

## Asynchronous and event-based
The definition of Reactive Extensions promises providing a way to construct asynchronous and event-based programs. From the perspective of UI programming, it is very important that the library provides a convenient way to decide to which synchronization context should your data be delivered. A lot of UI frameworks require you to access UI elements only from a specific UI thread.

Using Reactive Extensions, you can parametrize the concurrency of the stream using `Scheduler` class. For instance, you can declare that all of the data processing should be executed on a `TaskPool` thread, and then (after the whole processing) deliver final values directly to UI thread. This mechanism is very flexible, easy to maintain, and - last but not least - easy to test. Since the synchronization context for your operation is hidden by an abstraction layer of your `Scheduler`, you can easily mock it. That allows you to easily simulate the passage of time in your unit tests.

## Learn more
As we stated at the beginning, it is not possible do describe such a huge topic as Reactive Extensions library in a single article. In fact, we have barely scratched the surface in this introduction.

If you want to learn more, there are a lot of free resources available online. You should start with a [Introduction to Rx](http://www.introtorx.com), a free book covering all aspects of the library in a very accessible way. Another great learning material can be found on the [Reactive Extensions official website](http://reactivex.io/intro.html).

If you are not into reading the whole book on the topic right away, you can check out [the introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754). It's a great read, easy to follow, not too shallow nor too deep. This article is based on [RxJS](https://github.com/ReactiveX/RxJS) - a JavaScript flavor of the library - but this should not be a problem (some functions names may differ, but the concepts are exactly the same).

If you are more into video tutorials, check out the [Becoming a C# Time Lord](https://channel9.msdn.com/Events/TechEd/Australia/2013/DEV422) presentation by Joe Albahari. 

If you wish to see some examples of the library usage, there is no better place than [Rx 101 samples wiki](http://rxwiki.wikidot.com/101samples).

We also encourage you to play with the [Rx marbles](http://rxmarbles.com/) - an interactive website displaying how different Rx operators work.

Finally, there is [a huge list of tutorials and other learning materials](http://reactivex.io/tutorials.html) on the Reactive Extensions website. Feel free to browse!
