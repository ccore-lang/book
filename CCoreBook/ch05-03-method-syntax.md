## Method Syntax

*Methods* are similar to functions: they’re declared with their name, they can have parameters and a return value, and they contain some code that is run when they’re called from somewhere else. However, methods are different from functions in that they’re defined within the context of a class (or an enum or a trait object, which we cover in Chapters 6 and 17, respectively), and are declared without the `func` keyword.

### Defining Methods

Let’s change the `Area` function that has a `Rectangle` instance as a parameter and instead make an `Area` method defined on the `Rectangle` class, as shown in Listing 5-13:

<span class="filename">Filename: src/Main.cc</span>

```csharp
class Rectangle 
{
    mut uint Width;
    mut uint Height;

    uint Area() =>
        this.Width * this.Height;
}

func void Main() {
    var rect1 = Rectangle { Width = 30, Height = 50 };

    WriteLine(
        $"The area of the rectangle is {rect1.Area()} square pixels."
    );
}
```

<span class="caption">Listing 5-13: Defining an `Area` method on the `Rectangle` class</span>

To define the function within the context of `Rectangle`, we move the `Area` function within the `class` curly brackets and remove the `func` keyword. In `Main`, where we called the `Area` function and passed `rect1` as an argument, we instead use *method syntax* to call the `Area` method on our `Rectangle` instance. The method syntax goes after an instance: we add a dot followed by the method name, parentheses, and any arguments.

In the body of `Area`, we use the implicit reference variable `&this`. CCore knows that the type of `&this` is `Rectangle&` due to this method’s being inside the `class Rectangle` context. Note that here we don't use the `&` before `this`, because we are *derefencing* the `&this` reference to gain access to its fields. 

The `&this` reference is also the implicit receiver of method calls or field accesses from within the context of the class, so we can also write the `Area` method like this:

```csharp
uint Area() =>
    Width * Height; // direct field access
```

Methods can borrow `this` immutably as we’ve done here, or borrow `self` mutably.

We’ve chosen to define `Area` here immutable for the same reason we used `Rectangle&` and not `Rectangle^` in the function version: we just want to read the data in the class, not write to it. If we wanted to change the instance that we’ve called the method on as part of what the method does, we’d define the method as `mut uint Area()` and the implicit receiver would be the mutable reference `^this`.

The main benefit of using methods instead of functions, in addition to using method syntax, is for organization. We’ve put all the things we can do with an instance of a type in one `class` block rather than making future users of our code search for capabilities of `Rectangle` in various places in the library we provide.

> ### Where’s the `->` Operator?
>
> In C and C++, two different operators are used for calling methods: you use `.` if you’re calling a method on the object directly and `->` if you’re calling the method on a pointer to the object and need to dereference the pointer first. In other words, if `object` is a pointer, `object->something()` is similar to `(*object).something()`.
>
> CCore doesn’t have an equivalent to the `->` operator; instead, CCore has a feature called *automatic referencing and dereferencing*.
>
> This automatic referencing behavior works because methods have a clear receiver—the type of `this`. Given the mutability of the receiver (indicated by the presence or absence of the `mut` keyword) and name of a method, CCore can figure out definitively whether the method is reading (`&this`) or mutating (`^this`).

### Methods with More Parameters

Let’s practice using methods by implementing a second method on the `Rectangle` class. This time, we want an instance of `Rectangle` to take another instance of `Rectangle` and return `true` if the second `Rectangle` can fit completely within `this`; otherwise it should return `false`. That is, we want to be able to write the program shown in Listing 5-14, once we’ve defined the `CanHold` method:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func void Main() {
    var rect1 = Rectangle { Width = 30, Height = 50 };
    var rect2 = Rectangle { Width = 10, Height = 40 };
    var rect3 = Rectangle { Width = 60, Height = 45 };

    WriteLine($"Can rect1 hold rect2? {rect1.CanHold(&rect2)}");
    WriteLine($"Can rect1 hold rect3? {rect1.CanHold(&rect3)}");
}
```

<span class="caption">Listing 5-14: Using the as-yet-unwritten `CanHold` method</span>

And the expected output would look like the following, because both dimensions of `rect2` are smaller than the dimensions of `rect1` but `rect3` is wider than `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

We know we want to define a method, so it will be within the `class Rectangle` block. The method name will be `CanHold`, and it will take an immutable borrow of another `Rectangle` as a parameter. We can tell what the type of the parameter will be by looking at the code that calls the method: `rect1.CanHold(&rect2)` passes in `&rect2`, which is an immutable borrow to `rect2`, an instance of `Rectangle`. This makes sense because we only need to read `rect2` (rather than write, which would mean we’d need a mutable borrow), and we want `Main` to retain ownership of `rect2` so we can use it again after calling the `CanHold` method. The return value of `CanHold` will be a Boolean, and the implementation will check whether the width and height of `this` are both greater than the width and height of the other `Rectangle`, respectively. Let’s add the new `CanHold` method to the `class` block from Listing 5-13, shown in Listing 5-15:

<span class="filename">Filename: src/Main.cc</span>

```csharp
class Rectangle 
{
    mut uint Width;
    mut uint Height;

    uint Area() =>
        this.Width * this.Height;

    bool CanHold(Rectangle &other) =>
        this.Width > other.Width && this.Height > other.Height;
}
```

<span class="caption">Listing 5-15: Implementing the `CanHold` method on `Rectangle` that takes another `Rectangle` instance as a parameter</span>

When we run this code with the `Main` function in Listing 5-14, we’ll get our desired output. Methods can take multiple parameters that we add to the signature, and those parameters work just like parameters in functions.

### Associated Functions

Another useful feature of `class` blocks is that we’re allowed to define functions within `class` blocks that *don’t* take `this` as an implicit parameter and are declared with the `func` keyword. These are called *associated functions* because they’re associated with the class. They’re still functions, not methods, because they don’t have an instance of the class to work with.

Associated functions are often used for *factories* that will return a new instance of the class. For example, we could provide an associated function that would have one dimension parameter and use that as both width and height, thus making it easier to create a square `Rectangle` rather than having to specify the same value twice:

<span class="filename">Filename: src/Main.cc</span>

```csharp
class Rectangle 
{
    mut uint Width;
    mut uint Height;

    func Rectangle Square(uint size) =>
        Rectangle { Width = size, Height = size };
}
```

To call this associated function, we use the `.` syntax with the class name; `var sq = Rectangle.Square(3);` is an example. This function is namespaced by the class.

### `implement` Blocks

A class also can have, and sometimes *must* have, an `implement` block. `implement` blocks are useful to segregate the *interface* of a class from the *implementation* of the class. The interface of a class consists of only the members of the class that are accessible to the code that uses the class, and the methods in the interface part can be defined with only the signature of the method with their implementation defined in the `implement` block:

```csharp
class Rectangle  // interface part
{
    mut uint Width;
    mut uint Height;

    uint Area();
    bool CanHold(Rectangle &other);
}

implement  // implementation part
{
    uint Area() =>
        this.Width * this.Height;

    bool CanHold(Rectangle &other) =>
        this.Width > other.Width && this.Height > other.Height
}
```

If you define additional methods, fields, or other members like functions and constructors, only in the `implement` block but not appearing in the interface of the class, this members of the class are *not* accessible outside the class, making them affectively *private members* of the class. 

```csharp
class Rectangle 
{
    mut uint Width;
    mut uint Height;

    func Rectangle Square(uint size);  // public method
}

implement
{
    func Rectangle Square(uint size) =>
        New(size, size);  // OK, New used within Rectangle

    func Rectangle New(uint width, uint Height) =>  // private method
        Rectangle { Width = width, Height = height };
}

func void Main() {
    var square = Rectangle.Square(3); // OK
    var rect = Rectangle.New(5, 3);   // Error: method not accessible
}
```

We'll see more on *private* and *public* members in Chapter 10.

In the previous examples we defined the implementations of the methods `Area` and `CanHold` in the same interface part of the `Rectangle` class; this was possible because both methods only used the `Width` and `Height` fields, which are themselves declared in the interface part, and hence are public members. Even when a public method uses only public members, it's a good practice to separate the implementation to the `implement` block, unless the method has a simple body if so is desired.

### Standalone `implement` Blocks

Besides defining `implement` blocks as part of a `class` declaration, we can also define `implement` blocks for a class on their own, either in the same file of the class or in another file, like in this example:

<span class="filename">Filename: src/Shapes.cc</span>

```csharp
class Rectangle
{
    mut uint Width;
    mut uint Height;

    uint Area();
    bool CanHold(Rectangle &other);
}
```

<span class="filename">Filename: src/Shapes.Impl.cc</span>

```csharp
implement Rectangle
{
    uint Area() =>
        this.Width * this.Height;

    bool CanHold(Rectangle &other) =>
        this.Width > other.Width && this.Height > other.Height
}
```

<span class="caption">Listing 5-16: Rewriting Listing 5-15 using an `implement` block in another source file.</span>

Also, each class is allowed to have multiple standalone `implement` blocks. For example, Listing 5-15 is equivalent to the code shown in Listing 5-17, which has each method in its own `implement` block:

```csharp
class Rectangle
{
    mut uint Width;
    mut uint Height;

    uint Area();
    bool CanHold(Rectangle &other);
}

implement Rectangle
{
    uint Area() =>
        this.Width * this.Height;
}

implement Rectangle
{
    bool CanHold(Rectangle &other) =>
        this.Width > other.Width && this.Height > other.Height
}
```

<span class="caption">Listing 5-17: Rewriting Listing 5-15 using multiple `implement` blocks</span>

There’s no reason to separate these methods into multiple `implement` blocks here, as this is a simple example. In more complicated code, it's a good practice to place related class interface definitions in a single file, and separate their `implement` blocks to one o more files, so we can focus our attention to the API of our classes, or to the details of their implementations:

<span class="filename">Filename: src/Shapes.cc</span>

```csharp
class Rectangle
{
    mut uint Width;
    mut uint Height;

    uint Area();
}

class Circle
{
    mut uint Radius;

    uint Area();
}
```

<span class="filename">Filename: src/Rectangle.Impl.cc</span>

```csharp
implement Rectangle
{
    uint Area() =>
        this.Width * this.Height;
}
```

<span class="filename">Filename: src/Circle.Impl.cc</span>

```csharp
implement Circle
{
    uint Area() =>
        Pi * this.Radius ** 2;
}
```

<span class="caption">Listing 5-18: Collecting related `class` definitions in the same file</span>

We’ll see a differnt case in which multiple `implement` blocks are useful in Chapter 10 where we discuss generic types and traits.

## Summary

Classes let you create custom types that are meaningful for your domain. By using classes, you can keep associated pieces of data connected to each other and name each piece to make your code clear. Methods let you specify the behavior that instances of your classes have, and associated functions let you namespace functionality that is particular to your class without having an instance available.

But classes aren’t the only way you can create custom types: let’s turn to CCore’s enum feature to add another tool to your toolbox.
