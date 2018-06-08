## Control Flow

Deciding whether or not to run some code depending on if a condition is true and deciding to run some code repeatedly while a condition is true are basic building blocks in most programming languages. The most common constructs that let you control the flow of execution of CCore code are `if` expressions and loops.

### `if` Expressions

An `if` expression allows you to branch your code depending on conditions. You provide a condition and then state, “If this condition is met, run this block of code. If the condition is not met, do not run this block of code.”

Create a new project called *Branches* in your *projects* directory to explore the `if` expression. In the *src/Main.cc* file, input the following:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    int number = 3;

    if (number < 5) {
        WriteLine("condition was true");
    } 
    else {
        WriteLine("condition was false");
    }
}
```

All `if` expressions start with the keyword `if`, which is followed by a condition inside a pair of parenthesis. In this case, the condition checks whether or not the variable `number` has a value less than 5. The block of code we want to execute if the condition is true is placed immediately after the condition inside curly brackets. Blocks of code associated with the conditions in `if` expressions are sometimes called *arms*, just like the arms in `match` expressions that we discussed in the “Comparing the Guess to the Secret Number” section of Chapter 2.

Optionally, we can also include an `else` expression, which we chose to do here, to give the program an alternative block of code to execute should the condition evaluate to false. If you don’t provide an `else` expression and the condition is false, the program will just skip the `if` block and move on to the next bit of code.

Try running this code; you should see the following output:

```text
$ dotnet run
   Compiling Branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was true
```

Let’s try changing the value of `number` to a value that makes the condition `false` to see what happens:

```C#,ignore
int number = 7;
```

Run the program again, and look at the output:

```text
$ dotnet run
   Compiling Branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was false
```

It’s also worth noting that the condition in this code *must* be a `bool`. If the condition isn’t a `bool`, we’ll get an error. For example:

<span class="filename">Filename: src/Main.cc</span>

```C#,ignore
func void Main() {
    int number = 3;

    if (number) {
        WriteLine("number was three");
    }
}
```

The `if` condition evaluates to a value of `3` this time, and CCore throws an error:

```text
error[E0308]: mismatched types
 --> src/Main.cc:4:8
  |
4 |     if (number) {
  |         ^^^^^^ expected bool, found integral variable
  |
  = note: expected type `bool`
             found type `{int}`
```

The error indicates that CCore expected a `bool` but got an integer. Unlike languages such as Ruby and JavaScript, CCore will not automatically try to convert non-Boolean types to a Boolean. You must be explicit and always provide `if` with a Boolean as its condition. If we want the `if` code block to run only when a number is not equal to `0`, for example, we can change the `if` expression to the following:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    int number = 3;

    if (number != 0) {
        WriteLine("number was something other than zero");
    }
}
```

Running this code will print `number was something other than zero`.

#### Handling Multiple Conditions with `else if`

You can have multiple conditions by combining `if` and `else` in an `else if` expression. For example:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    int number = 6;

    if (number % 4 == 0) {
        WriteLine("number is divisible by 4");
    } 
    else if (number % 3 == 0) {
        WriteLine("number is divisible by 3");
    } 
    else if (number % 2 == 0) {
        WriteLine("number is divisible by 2");
    } 
    else {
        WriteLine("number is not divisible by 4, 3, or 2");
    }
}
```

This program has four possible paths it can take. After running it, you should
see the following output:

```text
$ dotnet run
   Compiling Branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
number is divisible by 3
```

When this program executes, it checks each `if` expression in turn and executes the first body for which the condition holds true. Note that even though 6 is divisible by 2, we don’t see the output `number is divisible by 2`, nor do we see the `number is not divisible by 4, 3, or 2` text from the `else` block. That’s because CCore only executes the block for the first true condition, and once it finds one, it doesn’t even check the rest.

As is the case with function bodies, the branches of an `if` expression can be a block of statements or a single expression. And as a single statement (as well as a statement block) is itself an expression evaluating to Void, which is also an expression in CCore, the previous example can be written:

```C#
func void Main() {
    int number = 6;

    if (number % 4 == 0)
        WriteLine("number is divisible by 4");
    else if (number % 3 == 0)
        WriteLine("number is divisible by 3");
    else if (number % 2 == 0)
        WriteLine("number is divisible by 2");
    else
        WriteLine("number is not divisible by 4, 3, or 2");
}
```

Using too many `else if` expressions can clutter your code, so if you have more than one, you might want to refactor your code. Chapter 6 describes a powerful CCore branching construct called `match` for these cases.

#### Using `if` in an Assigment Statement

Because `if` is an expression, we can use it on the right side of an assignment statement, as in Listing 3-2:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var condition = true;
    var number = if (condition) 5 else 6;

    WriteLine($"The value of number is: {number}");
}
```

<span class="caption">Listing 3-2: Assigning the result of an `if` expression to a variable</span>

The `number` variable will be bound to a value based on the outcome of the `if` expression. Run this code to see what happens:

```text
$ dotnet run
   Compiling Branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/branches`
The value of number is: 5
```

Remember that numbers by themselves are also expressions. In this case, the value of the whole `if` expression depends on which block of code executes. This means the values that have the potential to be results from each arm of the `if` must be the same type; in Listing 3-2, the results of both the `if` arm and the `else` arm were `int` integers. If the types are mismatched, as in the following example, we’ll get an error:

<span class="filename">Filename: src/Main.cc</span>

```C#,ignore
func void Main() {
    var condition = true;

    var number = if (condition) 5 else "six";

    WriteLine($"The value of number is: {number}");
}
```

When we try to compile this code, we’ll get an error. The `if` and `else` arms have value types that are incompatible, and CCore indicates exactly where to find the problem in the program:

```text
error[E0308]: if and else have incompatible types
 --> src/Main.cc:4:18
  |
4 |       var number = if (condition) 5 else "six";
  |                    ^_________________________^ 
  |                    expected integral variable, found string&
  |
  = note: expected type `{integer}`
             found type `string&`
```

The expression in the `if` block evaluates to an integer, and the expression in the `else` block evaluates to a string. This won’t work because variables must have a single type. CCore needs to know at compile time what type the `number` variable is, definitively, so it can verify at compile time that its type is valid everywhere we use `number`. Rust wouldn’t be able to do that if the type of `number` was only determined at runtime; the compiler would be more complex and would make fewer guarantees about the code if it had to keep track of multiple hypothetical types for any variable.

### Repetition with Loops

It’s often useful to execute a block of code more than once. For this task, CCore provides several *loops*. A loop runs through the code inside the loop body to the end and then starts immediately back at the beginning. To experiment with loops, let’s make a new project called *Loops*.

CCore has three kinds of loops: `while`, `until`, and `for`. Let’s try each one.

#### Repeating Code with `while`

The `while` keyword tells CCore to execute a block of code over and over again forever or until you explicitly tell it to stop.

As an example, change the *src/Main.cc* file in your *Loops* directory to look like this:

<span class="filename">Filename: src/Main.cc</span>

```C#,ignore
func void Main() {
    while {
        WriteLine("again!");
    }
}
```

When we run this program, we’ll see `again!` printed over and over continuously until we stop the program manually. Most terminals support a keyboard shortcut, <span class="keystroke">ctrl-c</span>, to halt a program that is stuck in a continual loop. Give it a try:

```text
$ dotnet run
   Compiling Loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29 secs
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

The symbol `^C` represents where you pressed <span class="keystroke">ctrl-c</span>. You may or may not see the word `again!` printed after the `^C`, depending on where the code was in the loop when it received the halt signal.

Fortunately, CCore provides another, more reliable way to break out of a loop. You can place the `break` keyword within the loop to tell the program when to stop executing the loop. Recall that we did this in the guessing game in the “Quitting After a Correct Guess” section of Chapter 2 to exit the program when the user won the game by guessing the correct number.

#### Conditional Loops with `while (condition)`

It’s often useful for a program to evaluate a condition within a loop. While the condition is true, the loop runs. When the condition ceases to be true, the program calls `break`, stopping the loop. This loop type could be implemented using a combination of `while`, `if`, `else`, and `break`; you could try that now in a program, if you’d like.

However, this pattern is so common that CCore has a built-in language construct for it, the same `while` loop but with a condition. Listing 3-3 uses `while`: the program loops three times, counting down each time, and then, after the loop, it prints another message and exits.

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    mut var number = 3;

    while (number != 0) {
        WriteLine($"{number}!");

        number = number - 1;
    }

    WriteLine("LIFTOFF!!!");
}
```

<span class="caption">Listing 3-3: Using a `while` loop to run code while a condition holds true</span>

This construct eliminates a lot of nesting that would be necessary if you used uncoinditioned `while`, `if`, `else`, and `break`, and it’s clearer. While a condition holds true, the code runs; otherwise, it exits the loop.

#### Looping Through a Collection with `for`

You could use the `while` construct to loop over the elements of a collection, such as an array. For example, let’s look at Listing 3-4:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var a = [10, 20, 30, 40, 50];
    mut int index = 0;

    while (index < 5) {
        WriteLine($"the value is: {a[index]}");
        index = index + 1;
    }
}
```

<span class="caption">Listing 3-4: Looping through each element of a collection using a `while` loop</span>

Here, the code counts up through the elements in the array. It starts at index `0`, and then loops until it reaches the final index in the array (that is, when `index < 5` is no longer true). Running this code will print every element in the array:

```text
$ dotnet run
   Compiling Loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

All five array values appear in the terminal, as expected. Even though `index` will reach a value of `5` at some point, the loop stops executing before trying to fetch a sixth value from the array.

But this approach is error prone; we could cause the program to abort if the index length is incorrect. It’s also slow, because the compiler adds runtime code to perform the conditional check on every element on every iteration through the loop.

As a more concise alternative, you can use a `for` loop and execute some code for each item in a collection. A `for` loop looks like this code in Listing 3-5:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var a = [10, 20, 30, 40, 50];

    for (var element in a) {
        WriteLine($"the value is: {element}");
    }
}
```

<span class="caption">Listing 3-5: Looping through each element of a collection using a `for` loop</span>

When we run this code, we’ll see the same output as in Listing 3-4. More importantly, we’ve now increased the safety of the code and eliminated the chance of bugs that might result from going beyond the end of the array or not going far enough and missing some items.

For example, in the code in Listing 3-4, if you removed an item from the `a` array but forgot to update the condition to `while index < 4`, the code would abort. Using the `for` loop, you wouldn’t need to remember to change any other code if you changed the number of values in the array.

The safety and conciseness of `for` loops make them the most commonly used loop construct in CCore. Even in situations in which you want to run some code a certain number of times, as in the countdown example that used a `while` loop in Listing 3-3, most programmers would use a `for` loop. The way to do that would be to use a `Range`, which is a type provided by the standard library that generates all numbers in sequence starting from one number and ending before another number.

Here’s what the countdown would look like using a `for` loop and another method we’ve not yet talked about, `Reverse`, to reverse the range:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    for (var number in (1..4).Reverse()) {
        WriteLine("{number}!");
    }
    WriteLine("LIFTOFF!!!");
}
```

This code is a bit nicer, isn’t it?

## Summary

You made it! That was a sizable chapter: you learned about variables, scalar and compound data types, functions, comments, `if` expressions, and loops! If you want to practice with the concepts discussed in this chapter, try building programs to do the following:

* Convert temperatures between Fahrenheit and Celsius.
* Generate the nth Fibonacci number.
* Print the lyrics to the Christmas carol “The Twelve Days of Christmas,”
taking advantage of the repetition in the song.

When you’re ready to move on, we’ll talk about a concept in CCore that *doesn’t* commonly exist in other programming languages: ownership.
