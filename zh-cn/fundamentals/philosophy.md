# Our Philosophy

This is a set of rules and guidelines standing behind the framework design choices.

## No setup

The setup of an app should be as easy as possible. Things should Just Work™ out of the box.

    TODO: "We have created a yeoman generator that will scaffold out
           an application with or without batteries. Your choice."

## Buffet Table

You can select only the pieces of the framework you like and build on them. You can mix ReactiveUI with other frameworks and libraries to your liking.

## Testability

You should be able to test your ViewModels. Stuff like schedulers and routing should not be an excuse for not writing unit tests. Framework should support mocking them easily.

* Becoming a time-lord: https://channel9.msdn.com/Events/TechEd/Australia/2013/DEV422

## Splat dependency

[Splat](https://github.com/paulcbetts/splat) is an amazing library specifically developed with mobile devices in mind. Don't be afraid to use it.



> "I generally think a lot of the advice around IoC/DI is pretty bad in the domain of 'cross-platform mobile applications', because you have to remember that a lot of their ideas were written for web apps, not mobile or desktop apps.
>
> For example, the vast majority of popular IoC containers concern themselves solely with resolution speed on a warm cache, while basically completely disregarding memory usage or startup time - this is 100% fine for server applications, because these things don't matter; but for a mobile app? Startup time is huge.
>
> Splat's Service Location solves a number of issues for RxUI:
>
> Service Location is fast, and has almost no overhead to set up.
It encapsulates several different common object lifetime models (i.e. 'create new every time', 'singleton', 'lazy'), just by writing the Func differently
It's Mono Linker friendly (generally)
Service Location allows us to register types in platform-specific code, but use them in PCL code."
>
> http://stackoverflow.com/a/26924067/496857