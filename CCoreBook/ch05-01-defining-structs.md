## Defining and Instantiating Classes

Classes are similar to tuples, which were discussed in Chapter 3. Like tuples, the pieces of a class can be different types. Unlike with tuples, you’ll name each piece of data so it’s clear what the values mean. As a result of these names, classes are more flexible than tuples: you don’t have to rely on the order of the data to specify or access the values of an instance.

To define a class, we enter the keyword `class` and name the entire class. A class’ name should describe the significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of the pieces of data, which we call *fields*. For example, Listing 5-1 shows a class that stores information about a user account:

```csharp
class User
{
    string Username;
    string Email;
    int SignInCount;
    bool Active;
}
```

<span class="caption">Listing 5-1: A `User` class definition</span>

To use a class after we’ve defined it, we create an *instance* of that class by specifying concrete values for each of the fields. We create an instance by stating the name of the class and then add curly brackets containing `key = value` pairs, where the keys are the names of the fields and the values are the data we want to store in those fields. We don’t have to specify the fields in the same order in which we declared them in the class. In other words, the class definition is like a general template for the type, and instances fill in that template with particular data to create values of the type. For example, we can declare a particular user as shown in Listing 5-2:

```csharp
class User 
{
    mut string Username;
    mut string Email;
    mut int SignInCount;
    mut bool Active;
}

var user1 = User {
    Email = $"someone@example.com",
    Username = $"someusername123",
    Active = true,
    SignInCount = 1,
};
```

<span class="caption">Listing 5-2: Creating an instance of the `User` class</span>

To get a specific value from a class, we can use dot notation. If we wanted just this user’s email address, we could use `user1.Email` wherever we wanted to use this value. If the instance is mutable, we can change a value by using the dot notation and assigning into a particular field. Listing 5-3 shows how to change the value in the `Email` field of a mutable `User` instance:

```csharp
class User 
{
    mut string Username;
    mut string Email;
    mut int SignInCount;
    mut bool Active;
}

mut var user1 = User {
    Email = $"someone@example.com",
    Username = $"someusername123",
    Active = true,
    SignInCount = 1,
};

user1.Email = $"anotheremail@example.com";
```

<span class="caption">Listing 5-3: Changing the value in the `Email` field of a `User` instance</span>

Note that the entire instance must be mutable.

As with any expression, we can construct a new instance of the class as the single expression in the function body to return that new instance. Listing 5-4 shows a `BuildUser` function that returns a `User` instance with the given email and username. The `Active` field gets the value of `true`, and the `SignInCount` gets a value of `1`.

```csharp
class User 
{
    mut string Username;
    mut string Email;
    mut int SignInCount;
    mut bool Active;
}

func User BuildUser(string email, string username) =>
    User {
        Email = email,
        Username = username,
        Active = true,
        SignInCount = 1,
    };
```

<span class="caption">Listing 5-4: A `BuildUser` function that takes an email and username and returns a `User` instance</span>

### Tuple Classes without Named Fields to Create Different Types

You can also define classes that look similar to tuples, called *tuple classes*. Tuple classes have the added meaning the class name provides but don’t have names associated with their fields; rather, they just have the types of the fields. Tuple classes are useful when you want to give the whole tuple a name and make the tuple be a different type than other tuples, and naming each field as in a regular class would be verbose or redundant.

To define a tuple class start with the `class` keyword and the class name followed by the types in the tuple. For example, here are definitions and usages of two tuple classes named `Color` and `Point`:

```csharp
class Color(int, int, int);
class Point(int, int, int);

var black = Color(0, 0, 0);
var origin = Point(0, 0, 0);
```

Note that the `black` and `origin` values are different types, because they’re instances of different tuple classes. Each class you define is its own type, even though the fields within the class have the same types. For example, a function that takes a parameter of type `Color` cannot take a `Point` as an argument, even though both types are made up of three `int` values. Otherwise, tuple class instances behave like tuples: you can destructure them into their individual pieces, you can use a `.` followed by the index to access an individual value, and so on.

### Void-Like Classes Without Any Fields

You can also define classes that don’t have any fields! These are called *void-like classes* because they behave similarly to the `void` type. Void-like classes can be useful in situations in which you need to implement a trait on some type but don’t have any data that you want to store in the type itself. We’ll discuss traits in Chapter 10.

> ### Ownership of Class Data
>
> In the `User` class definition in Listing 5-1, we used the owned `string` type rather than the `char[]&` string slice type. This is a deliberate choice because we want instances of this class to own all of its data and for that data to be valid for as long as the entire class is valid.
>
> It’s possible for classes to store references to data owned by something else, but to do so requires the use of *lifetimes*, a CCore feature that we’ll discuss in Chapter 10. Lifetimes ensure that the data referenced by a class is valid for as long as the class is. Let’s say you try to store a reference in a class without specifying lifetimes, like this, which won’t work:
>
> <span class="filename">Filename: src/Main.cc</span>
>
> ```csharp,ignore
> class User
> {
>     char[] &Username,
>     char[] &Email,
>     int SignInCount,
>     bool Active,
> }
>
> func void Main() {
>     mut var user1 = User {
>         &Username = "someusername123",
>         &Email = "someone@example.com",
>         Active = true,
>         SignInCount = 1,
>     };
> }
> ```
>
> The compiler will complain that it needs lifetime specifiers.
>
> In Chapter 10, we’ll discuss how to fix these errors so you can store > references in classes, but for now, we’ll fix errors like these using owned > types like `string` instead of references like `char[]&`.
