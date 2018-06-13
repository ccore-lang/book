## What Is Ownership?

CCore’s central feature is *ownership*. Although the feature is straightforward to explain, it has deep implications for the rest of the language.

All programs have to manage the way they use a computer’s memory while running. Some languages have garbage collection that constantly looks for no longer used memory as the program runs; in other languages, the programmer must explicitly allocate and free the memory. CCore uses a third approach: memory is managed with a garbage collector *and* through a system of ownership with a set of rules that the compiler checks at compile time. None of the ownership features slow down your program while it’s running.

Because ownership is a new concept for many programmers, it does take some time to get used to. The good news is that the more experienced you become with CCore and the rules of the ownership system, the more you’ll be able to naturally develop code that is safe and efficient. Keep at it!

When you understand ownership, you’ll have a solid foundation for understanding the features that make CCore unique. In this chapter, you’ll learn ownership by working through some examples that focus on a very common data structure: strings.

> ### The Stack and the Heap
>
> In many programming languages, you don’t have to think about the stack and the heap very often. But in a systems-like programming language like CCore, whether a value is on the stack or the heap has more of an effect on how the language behaves and why you have to make certain decisions. Parts of ownership will be described in relation to the stack and the heap later in this chapter, so here is a brief explanation in preparation.
>
> Both the stack and the heap are parts of memory that is available to your code to use at runtime, but they are structured in different ways. The stack stores values in the order it gets them and removes the values in the opposite order. This is referred to as *last in, first out*. Think of a stack of plates: when you add more plates, you put them on top of the pile, and when you need a plate, you take one off the top. Adding or removing plates from the middle or bottom wouldn’t work as well! Adding data is called *pushing onto the stack*, and removing data is called *popping off the stack*.
>
> The stack is fast because of the way it accesses the data: it never has to search for a place to put new data or a place to get data from because that place is always the top. Another property that makes the stack fast is that all data on the stack must take up a known, fixed size.
>
> Data with a size unknown at compile time or a size that might change can be stored on the heap instead. The heap is less organized: when you put data on the heap, you ask for some amount of space. The operating system finds an empty spot somewhere in the heap that is big enough, marks it as being in use, and returns a *pointer*, which is the address of that location. This process is called *allocating on the heap*, sometimes abbreviated as just “allocating.” Pushing values onto the stack is not considered allocating. Because the pointer is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you have to follow the pointer.
>
> Think of being seated at a restaurant. When you enter, you state the number of people in your group, and the staff finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.
>
> Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. Continuing the analogy, consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap). Allocating a large amount of space on the heap can also take time.
>
> When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.
>
> Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that ownership addresses. Once you understand ownership, you won’t need to think about the stack and the heap very often, but knowing that managing heap data is why ownership exists can help explain why it works the way it does.

### Ownership Rules

First, let’s take a look at the ownership rules. Keep these rules in mind as we work through the examples that illustrate them:

> 1. Each value in CCore has a variable that’s called its *owner*.
> 2. There can only be one owner at a time.
> 3. When the owner goes out of scope, the value will be dropped.

### Variable Scope

We’ve walked through an example of a CCore program already in Chapter 2. Now that we’re past basic syntax, we won’t include all the `func void Main() {` code in examples, so if you’re following along, you’ll have to put the following examples inside a `Main` function manually. As a result, our examples will be a bit more concise, letting us focus on the actual details rather than boilerplate code.

As a first example of ownership, we’ll look at the *scope* of some variables. A scope is the range within a program for which an item is valid. Let’s say we have a variable that looks like this:

```csharp
var s = $"hello";
```

The variable `s` refers to a string literal, where the value of the string is hardcoded into the text of our program. The variable is valid from the point at which it’s declared until the end of the current *scope*. Listing 4-1 has comments annotating where the variable `s` is valid:

```csharp
{                      // s is not valid here, it’s not yet declared
    var s = $"hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

<span class="caption">Listing 4-1: A variable and the scope in which it is valid</span>

In other words, there are two important points in time here:

* When `s` comes *into scope*, it is valid.
* It remains valid until it goes *out of scope*.

At this point, the relationship between scopes and when variables are valid is similar to that in other programming languages. Now we’ll build on top of this understanding by introducing the `string` type.

### The `string` Type

To illustrate the rules of ownership, we need a data type that is more complex than the ones we covered in the “Data Types” section of Chapter 3. The types covered previously are all stored on the stack and popped off the stack when their scope is over, but we want to look at data that is stored on the heap and explore how CCore knows when to clean up that data.

We’ll use `string` as the example here and concentrate on the parts of `string` that relate to ownership. These aspects also apply to other complex data types provided by the standard library and that you create. We’ll discuss `string` in more depth in Chapter 8.

We’ve already seen string literals, where a string value is hardcoded into our program. String literals are convenient, but they aren’t suitable for every situation in which we may want to use text. One reason is that they’re immutable. Another is that not every string value can be known when we write our code: for example, what if we want to take user input and store it? For these situations, CCore has a second string type, `string`. This type is allocated on the heap and as such is able to store an amount of text that is unknown to us at compile time. You can create a `string` from a string literal using a `string` constructor, like so:

```csharp
var s = string("hello");
```

This kind of string *can* be mutated:

```csharp
mut var s = string("hello");

s.Append(", world!"); // Append() appends a literal to a string

WriteLine($"{s}"); // This will print `hello, world!`
```

So, what’s the difference here? Why can `string` be mutated but literals cannot? The difference is how these two types deal with memory.

### Memory and Allocation

In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s immutability. Unfortunately, we can’t put a blob of memory into the binary for each piece of text whose size is unknown at compile time and whose size might change while running the program.

With the `string` type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents. This means:

* The memory must be requested from the operating system at runtime.
* We need a way of returning this memory to the operating system when we’re done with our `string`.

That first part is done by us: when we call `string()`, its implementation requests the memory it needs. This is pretty much universal in programming languages.

However, the second part is different. In languages with a *garbage collector (GC)*, the GC keeps track and cleans up memory that isn’t being used anymore, and we don’t need to think about it. Without a GC, it’s our responsibility to identify when memory is no longer being used and call code to explicitly clean it, just as we did to request it. Doing this correctly has historically been a difficult programming problem. If we forget, we’ll waste memory. If we do it too early, we’ll have an invalid variable. If we do it twice, that’s a bug too. We need to pair exactly one `allocate` with exactly one `free`.

CCore takes a different path: the memory is automatically cleaned once the variable that owns it goes out of scope. Here’s a version of our scope example from Listing 4-1 using a `string` instead of a string literal:

```csharp
{
    var s = string("hello"); // s is valid from this point forward

    // do stuff with s
}                                  // this scope is now over, and s is no
                                   // longer valid
```

There is a natural point at which we can clean the memory our `string` needs: when `s` goes out of scope. When a variable goes out of scope, CCore calls a special function for us. This function is a *destructor*, and it’s where the author of `string` can put the code to clean the memory. CCore calls the desturctor automatically at the closing `}`.

> Note: In C++, this pattern of deallocating resources at the end of an item’s lifetime is sometimes called *Resource Acquisition Is Initialization (RAII)*. The destructor in CCore will be familiar to you if you’ve used RAII patterns.

This pattern has a profound impact on the way CCore code is written. It may seem simple right now, but the behavior of code can be unexpected in more complicated situations when we want to have multiple variables use the data we’ve allocated on the heap. Let’s explore some of those situations.

Also, creating a `string` from a string literal is a pattern so common, that CCore has special syntax to create *string values*:

```csharp
var s = string("hello"); // explicit creation

var s = $"hello"; // equivalent syntax
```

#### Ways Variables and Data Interact: Move

Multiple variables can interact with the same data in different ways in CCore. Let’s look at an example using an integer in Listing 4-2:

```csharp
var x = 5;
var y = x;
```

<span class="caption">Listing 4-2: Assigning the integer value of variable `x` to `y`</span>

We can probably guess what this is doing: “bind the value `5` to `x`; then make a copy of the value in `x` and bind it to `y`.” We now have two variables, `x` and `y`, and both equal `5`. This is indeed what is happening, because integers are simple values with a known, fixed size, and these two `5` values are pushed onto the stack.

Now let’s look at the `string` version:

```csharp
var s1 = $"hello";
var s2 = s1;
```

This looks very similar to the previous code, so we might assume that the way it works would be the same: that is, the second line would make a copy of the value in `s1` and bind it to `s2`. But this isn’t quite what happens. And in fact, it doesn't even compile.

Take a look at Figure 4-1 to see what would happen to `string` under the covers in this situation. A `string` is made up of three parts, shown on the left: a pointer to the memory that holds the contents of the string, a length, and a capacity. This group of data is stored on the stack. On the right is the memory on the heap that holds the contents.

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-1: Representation in memory of a `string` holding the value `"hello"` bound to `s1`</span>

The length is how much memory, in bytes, the contents of the `string` is currently using. The capacity is the total amount of memory, in bytes, that the `string` has received from the operating system. The difference between length and capacity matters, but not in this context, so for now, it’s fine to ignore the capacity.

When we assign `s1` to `s2`, the `string` data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to. In other words, the data representation in memory looks like Figure 4-2.

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-2: Representation in memory of the variable `s2` that has a copy of the pointer, length, and capacity of `s1`</span>

The representation does *not* look like Figure 4-3, which is what memory would look like if CCore instead copied the heap data as well. If CCore did this, the operation `s2 = s1` could be very expensive in terms of runtime performance if the data on the heap were large.

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Another possibility for what `s2 = s1` might do if CCore copied the heap data as well</span>

Earlier, we said that when a variable goes out of scope, CCore automatically calls the destructor and cleans up the heap memory for that variable. But Figure 4-2 shows both data pointers pointing to the same location. This is a problem: when `s2` and `s1` go out of scope, they will both try to free the same memory. This is known as a *double free* error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, there’s one more detail to what happens in this situation in CCore. Instead of trying to copy the allocated memory, CCore *moves* it: considers `s1` to no longer be valid and, therefore, CCore doesn’t need to free anything when `s1` goes out of scope. Check out what happens when you try to use `s1` after `s2` is created; it won’t work:

```csharp,ignore
var s1 = $"hello";
var s2 = move s1;

WriteLine($"{s1}, world!");
```

You’ll get an error like this because CCore prevents you from using the invalidated reference:

```text
error[E0382]: use of moved value: `s1`
 --> src/Main.cc:5:28
  |
3 |     var s2 = move s1;
  |         -- value moved here
4 |
5 |     WriteLine($"{s1}, world!");
  |                  ^^ value used here after move
  |
```

If you’ve heard the terms *shallow copy* and *deep copy* while working with other languages, the concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. But because CCore also invalidates the first variable, instead of being called a shallow copy, it’s known as a *move*. Here we would read this by saying that `s1` was *moved* into `s2`. So what actually happens is shown in Figure 4-4.

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Representation in memory after `s1` has been invalidated</span>

That solves our problem! With only `s2` valid, when it goes out of scope, it alone will free the memory, and we’re done.

In addition, there’s a design choice that’s implied by this: CCore will never automatically create “deep” copies of your data. Therefore, any *automatic* copying can be assumed to be inexpensive in terms of runtime performance.

#### Ways Variables and Data Interact: copy

If we *do* want to deeply copy the heap data of the `string`, not just the stack data, we can use a common operator called `copy`.

Here’s an example of the `copy` operator in action:

```csharp
var s1 = $"hello";
var s2 = copy s1;

WriteLine($"s1 = {s1}, s2 = {s2}");
```

This works just fine and explicitly produces the behavior shown in Figure 4-3, where the heap data *does* get copied.

When you see a call to `copy`, you know that some arbitrary code is being executed and that code may be expensive. It’s a visual indicator that something different is going on.

#### Stack-Only Data: Copy constructor

There’s another wrinkle we haven’t talked about yet. This code using integers, part of which was shown earlier in Listing 4-2, works and is valid:

```csharp
var x = 5;
var y = x;

WriteLine($"x = {x}, y = {y}");
```

But this code seems to contradict what we just learned: we don’t have a call to `copy`, but `x` is still valid and wasn’t moved into `y`.

The reason is that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. That means there’s no reason we would want to prevent `x` from being valid after we create the variable `y`. In other words, there’s no difference between deep and shallow copying here, so calling `copy` wouldn’t do anything different from the usual shallow copying and we can leave it out.

CCore types can have a special constructor called the default `copy` constructor that is defined on types like integers that are stored on the stack (we’ll talk more about constructors in Chapter 10). If a type has a `copy` contructor, an older variable is still usable after assignment. CCore won’t define a default `copy` constructor if the type, or any of its parts, has defined a destructor. If the type needs something special to happen when the value goes out of scope and we try to copy it with the `copy` operator, we’ll get a compile time error. Unless we define an explicit `copy` constructor on our type like in the case of `string`, and in this case is mandatory to use the `copy` operator to copy the value.

So what types are `copy`? You can check the documentation for the given type to be sure, but as a general rule, any group of simple scalar values can be `copy`, and nothing that requires allocation or is some form of resource is `copy`. Here are some of the types that are `copy`:

* All the integer types, such as `uint`.
* The Boolean type, `bool`, with values `true` and `false`.
* All the floating point types, such as `double`.
* The character type, `char`.
* Tuples, but only if they contain types that are also `copy`. For example, `(int, int)` is `copy`, but `(int, string)` is not.

### Ownership and Functions

The semantics for passing a value to a function are similar to those for assigning a value to a variable. Passing a variable to a function will move or copy, just as assignment does. Listing 4-3 has an example with some annotations showing where variables go into and out of scope:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    string s = $"hello";     // s comes into scope

    TakesOwnership(move s);  // s's value moves into the function...
                             // ... and so is no longer valid here

    int x = 5;               // x comes into scope

    MakesCopy(x);            // x would move into the function,
                             // but int is copy, so it’s okay to still
                             // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, 
  // nothing special happens.

func void TakesOwnership(string someString) { // some_string comes into scope
    WriteLine(someString);
} // Here, someString goes out of scope and its destructor is called.
  // The backing memory is freed.

func void MakesCopy(int someInteger) { // some_integer comes into scope
    WriteLine(someInteger);
} // Here, someInteger goes out of scope. Nothing special happens.
```

<span class="caption">Listing 4-3: Functions with ownership and scope annotated</span>

If we tried to use `s` after the call to `TakesOwnership`, CCore would throw a compile time error. These static checks protect us from mistakes. Try adding code to `Main` that uses `s` and `x` to see where you can use them and where the ownership rules prevent you from doing so.

### Return Values and Scope

Returning values can also transfer ownership. Listing 4-4 is an example with similar annotations to those in Listing 4-3:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    var s1 = GivesOwnership();            // GivesOwnership moves its return
                                          // value into s1

    var s2 = $"hello";                    // s2 comes into scope

    var s3 = TakesAndGivesBack(move s2);  // s2 is moved into
                                          // TakesAndGivesBack, which also
                                          // moves its return value into s3
} // Here, s3 goes out of scope and is freed. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is freed.

func string GivesOwnership()           // GivesOwnership will move its
                                       // return value into the function
                                       // that calls it

    var someString = $"hello";         // someString comes into scope

    return someString;                 // someString is returned and
                                       // moves out to the calling
                                       // function.
}

// TakesAndGivesBack will take a string and return one.
func string TakesAndGivesBack(string aString) { // aString comes into
                                                // scope

    return aString;  // aString is returned and 
                     // moves out to the calling function
}
```

<span class="caption">Listing 4-4: Transferring ownership of return values</span>

Note that the `return` always moves its value out of the function, so we don't a `move` operator to return a value.

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by its destructor unless the data has been moved to be owned by another variable.

Taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? It’s quite annoying that anything we pass in also needs to be passed back if we want to use it again, in addition to any data resulting from the body of the function that we might want to return as well.

It’s possible to return multiple values using a tuple, as shown in Listing 4-5:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    var s1 = $"hello";

    (var s2, var len) = CalculateLength(s1);

    WriteLine($"The length of '{s2}' is {len}.");
}

func (string, int) CalculateLength(string s) {
    int length = s.Length; // Length returns the length of a String
    return (s, length);
}
```

<span class="caption">Listing 4-5: Returning ownership of parameters</span>

But this is too much ceremony and a lot of work for a concept that should be common. Luckily for us, CCore has a feature for this concept, called *references*.
