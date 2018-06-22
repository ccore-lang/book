## An Example Program Using Classes

To understand when we might want to use class, let’s write a program that calculates the area of a rectangle. We’ll start with single variables, and then refactor the program until we’re using classes instead.

Let’s make a new binary project called *Rectangles* that will take the width and height of a rectangle specified in pixels and calculate the area of the rectangle. Listing 5-8 shows a short program with one way of doing exactly that in our project’s *src/Main.cc*:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    uint width1 = 30;
    uint height1 = 50;

    WriteLine(
        $"The area of the rectangle is {Area(width1, height1)} square pixels."
    );
}

func uint Area(uint width, uint height) =>
    width * height;
```

<span class="caption">Listing 5-8: Calculating the area of a rectangle specified by separate width and height variables</span>

Now, run this program using `dotnet run`:

```text
The area of the rectangle is 1500 square pixels.
```

Even though Listing 5-8 works and figures out the area of the rectangle by calling the `Area` function with each dimension, we can do better. The width and the height are related to each other because together they describe one rectangle.

The issue with this code is evident in the signature of `Area`:

```csharp,ignore
func uint Area(uint width, uint height) =>
```

The `Area` function is supposed to calculate the area of one rectangle, but the function we wrote has two parameters. The parameters are related, but that’s not expressed anywhere in our program. It would be more readable and more manageable to group width and height together. We’ve already discussed one way we might do that in “The Tuple Type” section of Chapter 3: by using tuples.

### Refactoring with Tuples

Listing 5-9 shows another version of our program that uses tuples:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    var rect1 = (30, 50);

    WriteLine(
        $"The area of the rectangle is {Area(rect1)} square pixels."
    );
}

func uint Area((uint, uint) dimensions) =>
    dimensions.0 * dimensions.1;
```

<span class="caption">Listing 5-9: Specifying the width and height of the rectangle with a tuple</span>

In one way, this program is better. Tuples let us add a bit of structure, and we’re now passing just one argument. But in another way, this version is less clear: tuples don’t name their elements, so our calculation has become more confusing because we have to index into the parts of the tuple.

It doesn’t matter if we mix up width and height for the area calculation, but if we want to draw the rectangle on the screen, it would matter! We would have to keep in mind that `width` is the tuple index `0` and `height` is the tuple index `1`. If someone else worked on this code, they would have to figure this out and keep it in mind as well. It would be easy to forget or mix up these values and cause errors, because we haven’t conveyed the meaning of our data in our code.

### Refactoring with Classes: Adding More Meaning

We use classes to add meaning by labeling the data. We can transform the tuple we’re using into a data type with a name for the whole as well as names for the parts, as shown in Listing 5-10:

<span class="filename">Filename: src/Main.cc</span>

```csharp
class Rectangle 
{
    mut uint Width;
    mut uint Height;
}

func void Main() {
    var rect1 = Rectangle { Width = 30, Height = 50 };

    WriteLine(
        $"The area of the rectangle is {Area(&rect1)} square pixels."
    );
}

func uint Area(Rectangle &rectangle) =>
    rectangle.Width * rectangle.Height;
```

<span class="caption">Listing 5-10: Defining a `Rectangle` class</span>

Here we’ve defined a class and named it `Rectangle`. Inside the curly brackets, we defined the fields as `Width` and `Height`, both of which have type `uint`. Then in `Main`, we created a particular instance of `Rectangle` that has a width of 30 and a height of 50.

Our `Area` function is now defined with one parameter, which we’ve named `&rectangle`, whose type is an immutable borrow of a class `Rectangle` instance. As mentioned in Chapter 4, we want to borrow the class rather than take ownership of it. This way, `Main` retains its ownership and can continue using `rect1`, which is the reason we use the `&` in the function signature and where we call the function.

The `Area` function accesses the `Width` and `Height` fields of the `Rectangle` instance. Our function signature for `Area` now says exactly what we mean: calculate the area of `Rectangle`, using its `Width` and `Height` fields. This conveys that the width and height are related to each other, and it gives descriptive names to the values rather than using the tuple index values of `0` and `1`. This is a win for clarity.

### Adding Useful Functionality with Default Traits

It’d be nice to be able to print an instance of `Rectangle` while we’re debugging our program and see the values for all its fields. Listing 5-11 uses the `WriteLine` function as we have used in previous chapters:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
class Rectangle 
{
    mut uint Width;
    mut uint Height;
}

func void Main() {
    var rect1 = Rectangle { Width = 30, Height = 50 };

    WriteLine($"rect1 is {rect1}");
}
```

<span class="caption">Listing 5-11: Printing a `Rectangle` instance</span>

When we run this code, we’ll see the following output:

```text
rect1 is Rectangle { Width: 30, Height: 50 }
```

String values (`$"rect1 is {rect1}"` in this example) can do many kinds of formatting, and by default, curly brackets tell the string value to use formatting known as `Format`: output intended for direct end user consumption. The primitive types we’ve seen so far implement `Format` by default, because there’s only one way you’d want to show a `1` or any other primitive type to a user. But with classes, the way the output should be formatted is less clear because there are more formatting possibilities: Do you want commas or not? Do you want to print the curly brackets? Should all the fields be shown? Luckily, in CCore classes have a default implementation of `Format`. This automatically provided implementation enables us to print our class in a way that is useful for developers so we can see its value while we’re debugging our code.

Our `Area` function is very specific: it only computes the area of rectangles. It would be helpful to tie this behavior more closely to our `Rectangle` class, because it won’t work with any other type. Let’s look at how we can continue to refactor this code by turning the `Area` function into an `Area` *method* defined on our `Rectangle` type.
