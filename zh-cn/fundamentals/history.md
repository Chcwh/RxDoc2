# Historical background

Long ago, when computer programming first came to be, machines had to be programmed quite manually. If the technician entered the correct sequence of machine codes in the correct order, then the resulting program behavior would satisfy the business requirements.

Then we invented a higher-level programming language: C. And once more, if the programmer wrote the correct sequence of instructions in the correct order, then the resulting program behavior would satisfy the business requirements.

## The 1980's: A Revolution in Software

The 1980's saw the invention of a new way to write computer programs: object-oriented programming. The programmer would write code that manipulated the data belonging to re-usable "objects", and if the programmer wrote the correct instructions in the correct order, then the resulting program behavior would satisfy the business requirements.

After that, things got pretty boring.

Most modern programming today is basically imperative, meaning it models the traditional fetch-execute cycle of a CPU. Perform and instruction, fetch the next one. Perform that one, and so on. For decades, programmers have had to mould their brains to fit the paradigm of the CPU.

## Imperative Programming is Dangerous

When we rely on hoping that the behavior that emerges from a program is correct, and that reliance is based on nothing more than a programmer's correctness, then we can easily find ourselves in a sticky situation. We can try and mitigate the costs of imperative programming with things like unit tests or integration tests, but why mitigate the costs when there's a better way?

Instead of telling a computer how to do its job, why don't we just tell it what it's job is and let it figure the rest out? The bottom line is imperative programming is error-prone and relies too heavily on the infallibility of the programmer.

## There is a Better Way

Instead of imperative programming, we can use a new paradigm to structure our code. This paradigm is called [Functional Reactive Programming](functional-reactive-programming.html).
