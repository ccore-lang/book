## Variables and Mutability

As mentioned in Chapter 2, by default variables are immutable. This is one of many nudges CCore gives you to write your code in a way that takes advantage of the safety and easy concurrency that CCore offers. However, you still have the option to make your variables mutable. Let’s explore how and why CCore encourages you to favor immutability and why sometimes you might want to opt out.

When a variable is immutable, once a value is bound to a name, you can’t change that value. To illustrate this, let’s generate a new project called *variables* in your *projects* directory by using `dotnet new Variables`.

Then, in your new *Variables* directory, open *src/Main.cc* and replace its code with the following code that won’t compile just yet:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func void Main() {
    var x = 5;
    WriteLine($"The value of x is: {x}");
    x = 6;
    WriteLine($"The value of x is: {x}");
}
```

Save and run the program using `dotnet run`. You should receive an error message, as shown in this output:

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/Main.cc:4:5
  |
2 |     var x = 5;
  |         - first assignment to `x`
3 |     WriteLine($"The value of x is: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

This example shows how the compiler helps you find errors in your programs. Even though compiler errors can be frustrating, they only mean your program isn’t safely doing what you want it to do yet; they do *not* mean that you’re not a good programmer! Experienced programmers still get compiler errors.

The error indicates that the cause of the error is that you `cannot assign twice to immutable variable x`, because you tried to assign a second value to the immutable `x` variable.

It’s important that we get compile-time errors when we attempt to change a value that we previously designated as immutable because this very situation can lead to bugs. If one part of our code operates on the assumption that a value will never change and another part of our code changes that value, it’s possible that the first part of the code won’t do what it was designed to do. The cause of this kind of bug can be difficult to track down after the fact, especially when the second piece of code changes the value only *sometimes*.

In CCore, the compiler guarantees that when you state that a value won’t change, it really won’t change. That means that when you’re reading and writing code, you don’t have to keep track of how and where a value might change. Your code is thus easier to reason through.

But mutability can be very useful. Variables are immutable only by default; as you did in Chapter 2, you can make them mutable by adding `mut` in front of the variable declaration. In addition to allowing this value to change, `mut` conveys intent to future readers of the code by indicating that other parts of the code will be changing this variable value.

For example, let’s change *src/Main.cc* to the following:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    mut var x = 5;
    WriteLine($"The value of x is: {x}");
    x = 6;
    WriteLine($"The value of x is: {x}");
}
```

When we run the program now, we get this:

```text
$ dotnet run
   Compiling Variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/Variables`
The value of x is: 5
The value of x is: 6
```

We’re allowed to change the value that `x` binds to from `5` to `6` when `mut` is used. In some cases, you’ll want to make a variable mutable because it makes the code more convenient to write than if it had only immutable variables.

There are multiple trade-offs to consider in addition to the prevention of bugs. For example, in cases where you’re using large data structures, mutating an instance in place may be faster than copying and returning newly allocated instances. With smaller data structures, creating new instances and writing in a more functional programming style may be easier to think through, so lower performance might be a worthwhile penalty for gaining that clarity.

### Differences Between Variables and Constants

Being unable to change the value of a variable might have reminded you of another programming concept that most other languages have: *constants*. Like immutable variables, constants are values that are bound to a name and are not allowed to change, but there are a few differences between constants and variables.

First, you aren’t allowed to use `mut` with constants. Constants aren’t just immutable by default—they’re always immutable.

You declare constants using the `const` keyword instead of the `var` keyword, and the type of the value *must* be annotated. We’re about to cover types and type annotations in the next section, “Data Types,” so don’t worry about the details right now. Just know that you must always annotate the type.

Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.

The last difference is that constants may be set only to a constant expression, not the result of a function call or any other value that could only be computed at runtime.

Here’s an example of a constant declaration where the constant’s name is `MaxPoints` and its value is set to 100,000. (CCore’s constant naming convention is to use the first letter of each word in uppercase):

```csharp
const int MaxPoints = 100_000;
```

Constants are valid for the entire time a program runs, within the scope they were declared in, making them a useful choice for values in your application domain that multiple parts of the program might need to know about, such as the maximum number of points any player of a game is allowed to earn or the speed of light.

Naming hardcoded values used throughout your program as constants is useful in conveying the meaning of that value to future maintainers of the code. It also helps to have only one place in your code you would need to change if the hardcoded value needed to be updated in the future.
