## References and Borrowing

The issue with the tuple code in Listing 4-5 is that we have to return the `string` to the calling function so we can still use the `string` after the call to `CalculateLength`, because the `string` was moved into `CalculateLength`.

Here is how you would define and use a `CalculateLength` function that has a reference to an object as a parameter instead of taking ownership of the value:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    var s1 = $"hello";

    var len = CalculateLength(&s1);

    WriteLine($"The length of '{s1}' is {len}.");
}

func int CalculateLength(string &s) {
    return s.Length;
}
```

First, notice that all the tuple code in the variable declaration and the function return value is gone. Second, note that we pass `&s1` into `CalculateLength` and, in its definition, we take `string&` rather than `string`.

These ampersands are *references*, and they allow you to refer to some value without taking ownership of it. Figure 4-5 shows a diagram.

<img alt="&String s pointing at String s1" src="img/trpl04-05.svg" class="center" />

<span class="caption">Figure 4-5: A diagram of `string &s` pointing at `string
s1`</span>

> Note: The opposite of referencing by using `&` is *dereferencing*, which is accomplished omitting the reference operator `&`. We’ll see some uses of dereferencing operator in Chapter 8 and discuss details of dereferencing in Chapter 15.

Let’s take a closer look at the function call here:

```csharp
func int CalculateLength(string &s) {
    return s.Length;
}

var s1 = $"hello";

var len = CalculateLength(&s1);
```

The `&s1` syntax lets us create a reference that *refers* to the value of `s1` but does not own it. Because it does not own it, the value it points to will not be dropped when the reference goes out of scope.

Likewise, the signature of the function uses `&` to indicate that the type of the parameter `s` is a reference. Let’s add some explanatory annotations:

```csharp
func int CalculateLength(string &s) { // &s is a reference to a String
    return s.Length;
} // Here, &s goes out of scope. But because it does not have ownership
  // of what it refers to, nothing happens.
```

The scope in which the variable `&s` is valid is the same as any function parameter’s scope, but we don’t drop what the reference points to when it goes out of scope because we don’t have ownership. When functions have references as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.

We call having references as function parameters *borrowing*. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back.

So what happens if we try to modify something we’re borrowing? Try the code in Listing 4-6. Spoiler alert: it doesn’t work!

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func void Main() {
    var s = $"hello";

    Change(&s);
}

func void Change(string &someString) {
    someString.Append(", world");
}
```

<span class="caption">Listing 4-6: Attempting to modify a borrowed value</span>

Here’s the error:

```text
error[E0596]: cannot borrow immutable borrowed content `someString` as mutable
 --> error.rs:8:5
  |
7 | func void Change(string &someString) {
  |                  ------------------ use `mut string&` here to make mutable
8 |     someString.Append(", world");
  |     ^^^^^^^^^^ cannot borrow as mutable
```

Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.

### Mutable References

We can fix the error in the code from Listing 4-6 with just a small tweak:

<span class="filename">Filename: src/Main.cc</span>

```csharp
func void Main() {
    mut var s = $"hello";

    Change(^s);
}

func void Change(string ^someString) {
    someString.Append(", world");
}
```

First, we had to change `s` to be `mut`. Then we had to create a mutable reference with `^s` and accept a mutable reference with `string ^someString`.

But mutable references have one big restriction: you can only have one mutable reference to a particular piece of data in a particular scope. This code will fail:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
mut var s = $"hello";

var ^r1 = ^s;
var ^r2 = ^s;
```

Here’s the error:

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> Main.cc:5:19
  |
4 |     var ^r1 = ^s;
  |               -- first mutable borrow occurs here
5 |     var ^r2 = ^s;
  |               ^^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

This restriction allows for mutation but in a very controlled fashion. It’s something that new CCore programmers struggle with, because most languages let you mutate whenever you’d like.

The benefit of having this restriction is that CCore can prevent data races at compile time. A *data race* is similar to a race condition and happens when these three behaviors occur:

* Two or more pointers access the same data at the same time.
* At least one of the pointers is being used to write to the data.
* There’s no mechanism being used to synchronize access to the data.

Data races cause undefined behavior and can be difficult to diagnose and fix when you’re trying to track them down at runtime; CCore prevents this problem from happening because it won’t even compile code with data races!

As always, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not *simultaneous* ones:

```csharp
mut var s = $"hello";

{
    var ^r1 = ^s;
} // ^r1 goes out of scope here, so we can make 
  //a new reference with no problems.

var ^r2 = ^s;
```

A similar rule exists for combining mutable and immutable references. This code results in an error:

```csharp,ignore
mut var s = $"hello";

var &r1 = &s; // no problem
var &r2 = &s; // no problem
var ^r3 = ^s; // BIG PROBLEM
```

Here’s the error:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> borrow_thrice.rs:6:19
  |
4 |     var &r1 = &s; // no problem
  |                - immutable borrow occurs here
5 |     var &r2 = &s; // no problem
6 |     var ^r3 = ^s; // BIG PROBLEM
  |                ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Whew! We *also* cannot have a mutable reference while we have an immutable one. Users of an immutable reference don’t expect the values to suddenly change out from under them! However, multiple immutable references are okay because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

Even though these errors may be frustrating at times, remember that it’s the CCore compiler pointing out a potential bug early (at compile time rather than at runtime) and showing you exactly where the problem is. Then you don’t have to track down why your data isn’t what you thought it was.

### Dangling References

In languages with pointers, it’s easy to erroneously create a *dangling pointer*, a pointer that references a location in memory that may have been given to someone else, by freeing some memory while preserving a pointer to that memory. In CCore, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

Let’s try to create a dangling reference, which CCore will prevent with a compile-time error:

<span class="filename">Filename: src/Main.cc</span>

```csharp,ignore
func void Main() {
    var &referenceToNothing = &Dangle();
}

func string &Dangle() {
    var s = $"hello";
    return &s;
}
```

Here’s the error:

```text
error[E0106]: missing lifetime specifier
 --> dangle.rs:5:16
  |
5 | func string &Dangle() {
  |             ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
```

This error message refers to a feature we haven’t covered yet: *lifetimes*. We’ll discuss lifetimes in detail in Chapter 10. But, if you disregard the parts about lifetimes, the message does contain the key to why this code is a problem:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

Let’s take a closer look at exactly what’s happening at each stage of our `Dangle` code:

```csharp,ignore
func string &Dangle() { // Dangle returns a reference to a string

    var s = $"hello"; // s is a new string

    return &s; // we return a reference to the string, &s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

Because `s` is created inside `Dangle`, when the code of `Dangle` is finished, `s` will be deallocated. But we tried to return a reference to it. That means this reference would be pointing to an invalid `string` That’s no good! CCore won’t let us do this.

The solution here is to return the `string` directly:

```csharp
func string NoDangle() {
    var s = $"hello";
    return s;
}
```

This works without any problems. Ownership is moved out, and nothing is deallocated.

### The Rules of References

Let’s recap what we’ve discussed about references:

* At any given time, you can have *either* (but not both of) one mutable reference or any number of immutable references.
* References must always be valid.

Next, we’ll look at a different kind of reference: slices.
