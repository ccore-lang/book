## Data Types

Every value in CCore is of a certain *data type*, which tells CCore what kind of data is being specified so it knows how to work with that data. We’ll look at two data type subsets: scalar and compound.

Keep in mind that CCore is a *statically typed* language, which means that it must know the types of all variables at compile time. The compiler can usually infer what type we want to use based on the value and how we use it.

### Scalar Types

A *scalar* type represents a single value. CCore has four primary scalar types: integers, floating-point numbers, decimal (also known as fixed-point) numbers, Booleans, and characters. You may recognize these from other programming languages. Let’s jump into how they work in CCore.

#### Integer Types

An *integer* is a number without a fractional component. We used one integer type in Chapter 2, the `uint` type. This type declaration indicates that the value it’s associated with should be an unsigned integer that takes up 32 bits of space. Table 3-1 shows the built-in integer types in CCore. Each variant in the Signed and Unsigned columns (for example, `short`) can be used to declare the type of an integer value.

<span class="caption">Table 3-1: Integer Types in CCore</span>

| Length  | Signed   | Unsigned  |
|---------|----------|-----------|
| 8-bit   | `sbyte` | `byte`   |
| 16-bit  | `short` | `ushort` |
| 32-bit  | `int`   | `uint`    |
| 64-bit  | `long`  | `ulong`  |

Each variant can be either signed or unsigned and has an explicit size. *Signed* and *unsigned* refer to whether it’s possible for the number to be negative or positive—in other words, whether the number needs to have a sign with it (signed) or whether it will only ever be positive and can therefore be represented without a sign (unsigned). It’s like writing numbers on paper: when the sign matters, a number is shown with a plus sign or a minus sign; however, when it’s safe to assume the number is positive, it’s shown with no sign. Signed numbers are stored using two’s complement representation (if you’re unsure what this is, you can search for it online; an explanation is outside the scope of this book).

Each signed variant can store numbers from -(2<sup>n - 1</sup>) to 2<sup>n - 1</sup> - 1 inclusive, where *n* is the number of bits that variant uses. So a `sbyte` can store numbers from -(2<sup>7</sup>) to 2<sup>7</sup> - 1, which equals -128 to 127. Unsigned variants can store numbers from 0 to 2<sup>n</sup> - 1, so a `byte` can store numbers from 0 to 2<sup>8</sup> - 1, which equals 0 to 255.

You can write integer literals in any of the forms shown in Table 3-2. Note that all number literals except the byte literal allow a type suffix, such as `57u`, and `_` as a visual separator, such as `1_000`.

<span class="caption">Table 3-2: Integer Literals in CCore</span>

| Number literals  | Example       |
|------------------|---------------|
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |

So how do you know which type of integer to use? If you’re unsure, CCore’s defaults are generally good choices, and integer types default to `int`: this type is generally the fastest, even on 64-bit systems.

##### Integer Overflow

Let's say that you have a `byte`, which can hold values between zero and `255`. What happens if you try to change it to `256`? This is called "integer overflow", and CCore has some interesting rules around this behavior. When compiling in debug mode, CCore checks for this kind of issue and will cause your program to *abort*, which is the term CCore uses when a program exits with an error. We'll discuss aborts more in Chapter 9.

In release builds, CCore does not check for overflow, and instead will do something called "two's complement wrapping." In short, `256` becomes `0`, `257` becomes `1`, etc. Relying on overflow is considered an error, even if this behavior happens.

#### Floating-Point Types

CCore also has two primitive types for *floating-point numbers*, which are numbers with decimal points. CCore’s floating-point types are `float` and `double`, which are 32 bits and 64 bits in size, respectively. The default type is `double` because on modern CPUs it has roughly more speed than `float`.

Here’s an example that shows floating-point numbers in action:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var x = 2.0; // double
    var y = 3.0f; // float
}
```

Floating-point numbers are represented according to the IEEE-754 standard. The `float` type is a single-precision float, and `double` has double precision.

#### Numeric Operations

CCore supports the basic mathematical operations you’d expect for all of the number types: addition, subtraction, multiplication, division, and remainder. The following code shows how you’d use each one in a variable declaration statement:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    // addition
    int sum = 5 + 10;

    // subtraction
    double difference = 95.5 - 4.3;

    // multiplication
    int product = 4 * 30;

    // division
    double quotient = 56.7 / 32.2;

    // remainder
    int remainder = 43 % 5;
}
```

Each expression in these statements uses a mathematical operator and evaluates to a single value, which is then bound to a variable. Appendix B contains a list of all operators that CCore provides.

#### The Boolean Type

As in most other programming languages, a Boolean type in CCore has two possible values: `true` and `false`. The Boolean type in CCore is specified using `bool`. For example:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var t = true;
    bool f = false; // with explicit type annotation
}
```

The main way to consume Boolean values is through conditionals, such as an `if` expression. We’ll cover how `if` expressions work in CCore in the “Control Flow” section.

Booleans are one byte in size.

#### The Character Type

So far we’ve worked only with numbers, but CCore supports letters too. CCore’s `char` type is the language’s most primitive alphabetic type, and the following code shows one way to use it. (Note that the `char` literal is specified with single quotes, as opposed to string literals, which use double quotes.)

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    char c = 'z';
    char z = 'ℤ';
    char heartEyedCat = '😻';
}
```

CCore’s `char` type represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII. Accented letters; Chinese, Japanese, and Korean characters; emoji; and zero-width spaces are all valid `char` values in CCore. Unicode Scalar Values range from `U+0000` to `U+D7FF` and `U+E000` to `U+10FFFF` inclusive. However, a “character” isn’t really a concept in Unicode, so your human intuition for what a “character” is may not match up with what a `char` is in CCore. We’ll discuss this topic in detail in “Strings” in Chapter 8.

### Compound Types

*Compound types* can group multiple values into one type. CCore has two primitive compound types: tuples and arrays.

#### The Tuple Type

A tuple is a general way of grouping together some number of other values with a variety of types into one compound type. Tuples have a fixed length: once declared, they cannot grow or shrink in size.

We create a tuple by writing a comma-separated list of values inside parentheses. Each position in the tuple has a type, and the types of the different values in the tuple don’t have to be the same. We’ve added optional type annotations in this example:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    (int, double, byte) tup = (500, 6.4, 1);
}
```

The variable `tup` binds to the entire tuple, because a tuple is considered a single compound element. To get the individual values out of a tuple, we can use pattern matching to destructure a tuple value, like this:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var tup = (500, 6.4, 1);
    (var x, var y, var z) = tup;
    WriteLine($"The value of y is: {y}");
}
```

This program first creates a tuple and binds it to the variable `tup`. It then uses a pattern to take `tup` and turn it into three separate variables, `x`, `y`, and `z`. This is called *destructuring*, because it breaks the single tuple into three parts. Finally, the program prints the value of `y`, which is `6.4`.

In addition to destructuring through pattern matching, we can access a tuple element directly by using a period (`.`) followed by the index of the value we want to access. For example:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    (int, double, byte) x = (500, 6.4, 1);
    var fiveHundred = x.0;
    var sixPointFour = x.1;
    var one = x.2;
}
```

This program creates a tuple, `x`, and then makes new variables for each element by using their index. As with most programming languages, the first index in a tuple is 0.

#### The Array Type

Another way to have a collection of multiple values is with an *array*. Unlike a tuple, every element of an array must have the same type. Arrays in CCore are different from arrays in some other languages because arrays in CCore have a fixed length, like tuples.

In CCore, the values going into an array are written as a comma-separated list inside square brackets:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var a = [1, 2, 3, 4, 5];
}
```

Arrays are useful when you want your data allocated on the stack rather than the heap (we will discuss the stack and the heap more in Chapter 4), or when you want to ensure you always have a fixed number of elements. An array isn’t as flexible as the list type, though. A list is a similar collection type provided by the standard library that *is* allowed to grow or shrink in size. If you’re unsure whether to use an array or a list, you should probably use a list. Chapter 8 discusses lists in more detail.

An example of when you might want to use an array rather than a list is in a program that needs to know the names of the months of the year. It’s very unlikely that such a program will need to add or remove months, so you can use an array because you know it will always contain 12 items:

```C#
var months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Arrays have an interesting type; it looks like this: `type[number]`. For example:

```C#
int[5] a = [1, 2, 3, 4, 5];
```

First, there's type of each element of the array. Since all elements have the same type, we only need to list it once. After the elements type, there's a number inside square brackets that indicates the length of the array. Since an array has a fixed size, this number is always the same, even if the array's elements are modified, it cannot grow or shrink.

##### Accessing Array Elements

An array is a single chunk of memory allocated on the stack. You can access elements of an array using indexing, like this:

<span class="filename">Filename: src/Main.cc</span>

```C#
func void Main() {
    var a = [1, 2, 3, 4, 5];

    int first = a[0];
    int second = a[1];
}
```

In this example, the variable named `first` will get the value `1`, because that is the value at index `[0]` in the array. The variable named `second` will get the value `2` from index `[1]` in the array.

##### Invalid Array Element Access

What happens if you try to access an element of an array that is past the end of the array? Say you change the example to the following code, which will compile but exit with an error when it runs:

<span class="filename">Filename: src/Main.cc</span>

```C#,ignore
func void Main() {
    var a = [1, 2, 3, 4, 5];
    int index = 10;

    int element = a[index];

    WriteLine($"The value of element is: {element}");
}
```

Running this code using `dotnet run` produces the following result:

```text
$ dotnet run
   Compiling Arrays v0.1.0 (file:///projects/arrays)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/arrays`
thread '<main>' aborted at 'index out of bounds: the len is 5 but the index is
 10', src/Main.cc:6
```

The compilation didn’t produce any errors, but the program resulted in a *runtime* error and didn’t exit successfully. When you attempt to access an element using indexing, CCore will check that the index you’ve specified is less than the array length. If the index is greater than the length, CCore will abort.

This is the first example of CCore’s safety principles in action. In many low-level languages, this kind of check is not done, and when you provide an incorrect index, invalid memory can be accessed. CCore protects you against this kind of error by immediately exiting instead of allowing the memory access and continuing. Chapter 9 discusses more of CCore’s error handling.
