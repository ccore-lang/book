## Functions

Functions are pervasive in CCore code. You’ve already seen one of the most important functions in the language: the `Main` function, which is the entry point of many programs. You’ve also seen the `func` keyword, which allows you to declare new functions.

CCore code uses *Pascal case* as the conventional style for function. In Pascal case, the first letter of each word is uppercase and all other letters are lowercase. Here’s a program that contains an example function definition:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    WriteLine("Hello, world!");

    AnotherFunction();
}

func void AnotherFunction() {
    WriteLine("Another function.");
}
```

Function definitions in CCore start with `func` and have a set of parentheses after the return type and function name. The curly brackets tell the compiler where the function body begins and ends.

We can call any function we’ve defined by entering its name followed by a set of parentheses. Because `AnotherFunction` is defined in the program, it can be called from inside the `Main` function. Note that we defined `AnotherFunction` *after* the `Main` function in the source code; we could have defined it before as well. CCore doesn’t care where you define your functions, only that they’re defined somewhere.

Let’s start a new binary project named *Functions* to explore functions further. Place the `AnotherFunction` example in *src/Main.cc* and run it. You should see the following output:

```text
$ dotnet run
   Compiling Functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28 secs
     Running `target/debug/Functions`
Hello, world!
Another function.
```

The lines execute in the order in which they appear in the `Main` function. First, the “Hello, world!” message prints, and then `AnotherFunction` is called and its message is printed.

### Function Parameters

Functions can also be defined to have *parameters*, which are special variables that are part of a function’s signature. When a function has parameters, you can provide it with concrete values for those parameters. Technically, the concrete values are called *arguments*, but in casual conversation, people tend to use the words *parameter* and *argument* interchangeably for either the variables in a function’s definition or the concrete values passed in when you call a function.

The following rewritten version of `AnotherFunction` shows what parameters look like in CCore:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    AnotherFunction(5);
}

func void AnotherFunction(int x) {
    WriteLine($"The value of x is: {x}");
}
```

Try running this program; you should get the following output:

```text
$ dotnet run
   Compiling Functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21 secs
     Running `target/debug/Functions`
The value of x is: 5
```

The declaration of `AnotherFunction` has one parameter named `x`. The type of `x` is specified as `int`. When `5` is passed to `AnotherFunction`, the `WriteLine` function puts `5` where the `x` inside the pair of curly brackets were in the format string.

In function signatures, you *must* declare the type of each parameter. This is a deliberate decision in CCore’s design: requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what you mean.

When you want a function to have multiple parameters, separate the parameter declarations with commas, like this:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    AnotherFunction(5, 6);
}

func void AnotherFunction(int x, int y) {
    WriteLine($"The value of x is: {x}");
    WriteLine($"The value of y is: {y}");
}
```

This example creates a function with two parameters, both of which are `int` types. The function then prints the values in both of its parameters. Note that function parameters don’t all need to be the same type, they just happen to be in this example.

Let’s try running this code. Replace the program currently in your *Functions* project’s *src/Main.cc* file with the preceding example and run it using `dotnet run`:

```text
$ dotnet run
   Compiling Functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/Functions`
The value of x is: 5
The value of y is: 6
```

Because we called the function with `5` as the value for  `x` and `6` is passed as the value for `y`, the two strings are printed with these values.

### Function Bodies

Function bodies are made up of a block of statements or optionally a single expression. So far, we’ve only covered functions with statement blocks, but you have seen an expression as part of statements. Because CCore is an expression-based language, this is an important distinction to understand. Other languages don’t have the same distinctions, so let’s look at what statements and expressions are and how their differences affect the bodies of functions.

### Statements and Expressions

We’ve actually already used statements and expressions. *Statements* are instructions that perform some action and do not return a value. *Expressions* evaluate to a resulting value. Let’s look at some examples.

Creating a variable and assigning a value to it is a statement. In Listing 3-1, `var y = 6;` is a statement:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    var y = 6;
}
```

<span class="caption">Listing 3-1: A `Main` function declaration containing one statement</span>

Function definitions are also statements; the entire preceding example is a statement in itself.

Statements do not return values. Therefore, you can’t assign an assignment statement to another variable, as the following code tries to do; you’ll get an error:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func Main() {
    var x = (var y = 6);
}
```

When you run this program, the error you’ll get looks like this:

```text
$ dotnet run
   Compiling Functions v0.1.0 (file:///projects/functions)
error: expected expression, found statement (`var`)
 --> src/Main.cc:2:14
  |
2 |     var x = (var y = 6);
  |              ^^^
  |
```

The `var y = 6` statement does not return a value, so there isn’t anything for `x` to bind to. This is different from what happens in other languages, such as C and Ruby, where the assignment returns the value of the assignment. In those languages, you can write `x = y = 6` and have both `x` and `y` have the value `6`; that is not the case in CCore.

Expressions evaluate to something and make up most of the rest of the code that you’ll write in CCore. Consider a simple math operation, such as `5 + 6`, which is an expression that evaluates to the value `11`. Expressions can be part of statements: in Listing 3-1, the `6` in the statement `var y = 6;` is an expression that evaluates to the value `6`. Calling a function is an expression.

### Functions with Return Values

Functions can return values to the code that calls them. We don’t name return values, but we do declare their type before the function name. You can return from a function by using the `return` keyword and specifying a value, or when the body of the function consist of a single expression, specifying the expression after an arrow (`=>`). Here’s an example of two functions that return a value, the first using a `return` statement and the second using an arrow expression:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func int Five() {
    return 5;
}

func int Three() => 3;

func void Main() {
    int x = Five();
    int y = Three();

    WriteLine($"The value of x + y is: {x + y}");
}
```

There are no function calls, or even statements in the `Three` function—just the number `3` by itself. That’s a perfectly valid function in CCore. Note that the function’s return type is specified, too, as `int`. Try running this code; the output should look like this:

```text
$ dotnet run
   Compiling Functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/functions`
The value of x + y is: 8
```

The `5` in `Five` is the function’s return value, which is why the return type is `int`. Let’s examine this in more detail. There are two important bits: first, the line `int x = Five();` shows that we’re using the return value of a function to initialize a variable. Because the function `Five` returns a `5`, that line is the same as the following:

```csharp
int x = 5;
```

Second, the `Three` function has no parameters and defines the type of the return value, but the body of the function is a lonely `3` because it’s an expression whose value we want to return.

Let’s look at another example:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    int x = PlusOne(5);

    WriteLine($"The value of x is: {x}");
}

func int PlusOne(int x) => 
    x + 1;
```

Running this code will print `The value of x is: 6`. But if we place a the expression `x + 1` as part of a statement, changing it from an expression to a statement, we’ll get an error.

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func void Main() {
    int x = PlusOne(5);

    WriteLine($"The value of x is: {x}");
}

func int PlusOne(x: i32) =>
    WriteLine(x + 1);
```

Compiling this code produces an error, as follows:

```text
error[E0308]: mismatched types
 --> src/Main.cc:7:28
  |
7 |   func int PlusOne(int x) =>
  |  __________________________^
8 | |     WriteLine(x + 1);
  | |_^ expected int, found void
  |
  = note: expected type `int`
             found type `void`
```

The main error message, “mismatched types,” reveals the core issue with this code. The definition of the function `PlusOne` says that it will return an `int`, but statements don’t evaluate to a value, which is expressed by the value `Void` (of type `void`). Therefore, nothing is returned, which contradicts the function definition and results in an error. In this output, CCore provides a message to possibly help rectify this issue.
