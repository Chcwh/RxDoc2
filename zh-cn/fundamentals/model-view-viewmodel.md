# Model-View-ViewModel

Model-View-ViewModel (MVVM for short) is usually described as:

> a design pattern that is a variation of Martin Fowler's Presentation Model pattern which abstracts a view's state and behavior making view independent of any specific UI framework.

Since you are reading this document, chances are that above explanation is not all that helpful. **So, let us start again**.

Model-View-ViewModel is a way to structure your application. It is an idea which helps you organize your classes and interactions between them. Note that MVVM is **not** a specific library or an API. Instead, it can be described as a [*way to define the architecture of an application*](http://wp.qmatteoq.com/the-mvvm-pattern-introduction/). 

There are lots of libraries and frameworks which use MVVM, or help you shape your program architecture in a MVVM manner. One of them is of course [ReactiveUI](http://reactiveui.net/). There are others, like for instance [MvvmCross](http://mvvmcross.com/), [MvvmLight](http://www.mvvmlight.net/), [Caliburn.Micro](http://caliburnmicro.com/) as well as [tons of others](https://www.nuget.org/packages?q=mvvm). All those libraries have one common goal - their purpose is to help you to split your UI and business code into maintainable pieces.

The name of the Model-View-ViewModel pattern comes from three layers of an application. You can imagine them as a three separate bags, containing the bits of code from your application. In practice, this would mean the bits of code belonging to these layers are being stored inside different folders, namespaces, or even projects.

## View

Let's start with the most obvious one of the three. What belongs to the *View*? The answer is pretty simple - everything that is given to you by the specific UI framework you work with. 

All the user interface controls belong to the *View* layer. That includes buttons, input boxes, alert pop-ups, and that fancy calendar widget you hacked around recently.

Note that in MVVM world, the word *View* has two meanings. We already mentioned the first one, which is a layer of your application. But *View* can also mean **a class that groups several logically connected user controls**.

This could be for example a `Page` class, if you develop in Xamarin. In WPF, this would be a `Window` class. For WinForms the class is called `Form`. You usually inherit from these classes and fill them with buttons, inputs and other useful elements. *View* classes' names usually correspond to their  functionality - like `LoginView` or `OrderCheckoutView`.

The important attribute of the *View* class is the fact that usually **it cannot be unit tested**.

Most UI frameworks have applied almost zero thought to unit testing when they were designed, or those concerns were deemed as out-of-scope. As a result, UI objects are often very difficult to test in unit test runners, as they are not just plain objects. They may have dependencies on a runloop existing, or often expect static classes / globals to be initialized in a certain way.

So, since UI classes are untestable, our new goal is to put as much of our interesting code into **a class that represents the *View*, but is just a regular class we can create**. Then, we want the actual code in the *View* to be as boring, mechanical, and as short as possible, because it is inherently untestable.

This helpful class is called a *ViewModel*.

## ViewModel
The somewhat confusing name of the *ViewModel* class comes from a fact that it is a *Model* of a *View*. This means, that you usually have one *ViewModel* per *View*. This doesn't have to be strictly true, but it is generally the case. For example, you will have a `LoginViewModel` for your `LoginView` etc.

*ViewModel* classes expose data and actions that can be executed using UI of an application. Data is exposed to the *View* via public properties. The actions are simply methods of a *ViewModel*, wrapped in a so-called [`Command` classes](commands/index.md). Generally, commands allow you to enforce when the associated method of a *ViewModel* can be executed.

The data and the commands are used by a *View* class associated with the *ViewModel*. Properties are read and written (e.g. in a textbox), while commands are usually executed when a user pushes a button. The mechanism that helps keep *View* and *ViewModel* state in sync is called [data binding](binding/index.md).

Another important aspect of *ViewModels* is that they are an abstraction to separate policy from mechanism. *ViewModels* do not deal in the specifics of Buttons and Menus and TextBoxes, they only describe how the data in these elements are related. For example, the *Copy* command has no direct knowledge of the MenuItem or the Button that it is connected to, it only models the *Action of Copying*. The View has the responsibility of mapping the *Copy* command to the controls that invoke it.

Note that it is the *View* class which holds a reference to the *ViewModel*, not the other way round. In fact, the *View* is free to be very tightly bound to the *ViewModel*. 

The fact that **the *ViewModel* does not reference the *View*** has profound consequences. Because *ViewModels* do not explicitly reference UI frameworks or controls, they can be **reused across platforms**. For instance, you can use the same *ViewModels* for your Xamarin and WPF application.


All the *ViewModel* classes combined constitute a *ViewModel* layer of you application.

## Model
The last layer of you application is called a *Model*. Once you know what *View* and *ViewModel* is all about, this one is simple to define. Basically, it is all that is left. This includes, but is not limited to:
- configuration classes
- REST API wrappers
- database repositiories
- etc.

*Model* classes usually serve as a source of data for your ViewModels. They are often injected in the constructor.

## Additional reading
There are lots of good introductory material to the MVVM pattern around the web. If you need more information, check out the [great introductory article written by qmatteoq](http://wp.qmatteoq.com/the-mvvm-pattern-introduction/). You will also enjoy [this entertaining piece by Jeremiah Morrill ](http://jmorrill.hjtcentral.com/Home/tabid/428/EntryId/433/Anatomy-of-an-MVVM-Application-or-How-Tards-Like-Me-Make-MVVM-Apps.aspx).

## All other articles on MVVM I have seen had diagrams included. Can I get one?

Sure, here you go.

![](https://i.stack.imgur.com/yDjEr.png)
