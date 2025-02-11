+++
title = 'Learning_rust2'
date = 2025-02-06T20:58:41-07:00
type = 'post'
+++

This is adapted from the BYU CS 465 website which can be found [here](https://cs465.zappala.org/winter2025/notes/learning-rust-2/)

I will be going in a slightly different order than the above webpage, but I think it will be infomative to all.

## Owned vs Unowned Values
Due to Rust's ownership rules, there are many types that represent these rules. Here are a few of them.

|Name              |Unowned| Owned  |
|------------------|-------|--------|
|Reference (Borrow)|`&T`   |`Box<T>`,`Rc<T>`,`Arc<T>`|
|Slice (Array)     |`&[T]` or `[T;n]` |`Vec<T>`|
|UTF-8 Character String|`&str` |`String`  |

Many owned values contain heap allocated values but will automatically clean up their resources when they go out of scope.

You can read more about `Box<T>` and `Rc<T>` in chapter 15 which is about smart pointers.

I'll introduce `Box<T>` it is a way of allocating things on the heap.
This is commonly used for making recursive data (i.e. putting a struct inside of itself).

## Vectors and Strings

### Vectors
Rust has the vector type or `Vec` that represents owned, resizable slices.
Since this is a type of slice, Rust allows for references of vectors (`&Vec<T>`) to fit into the same arguments as `&[T]`.
This is especially useful when writing an APIs that take slices by reference and does something to them.

You can create a vector quickly by using the `vec!` macro as shown below:
```rust
let v1 = vec![1, 2, 3, 4, 5];
let v2 = vec![1; 10]; // Creates a Vec of 10 1s

```
Here are some useful Vec methods and usages:
```rust
let mut v = Vec::new();

v.push(2)
v.pop()
v.extend_from_slice(&[1, 2, 3, 4]);

if v.len() == 0 {
    println!("Empty Vec");
}
```

#### Iterators
Rust likes to use iterators to provide a powerful functional interface on data.
The easiest ways to get an iterator is to call one of these functions, `iter`, `iter_mut`, or `into_iter`.
The last one takes ownership of the vector and turns it into a iterator type.
Iterators have a whole bunch of useful methods to transform the stream of data.
Below is an example of using `map` to alter the data and collect to convert the iterator back into a vector.
```rust
v = v.into_iter().map(|x| x + 1).collect::<Vec<_>>();
```

It should be noted that when passing a vector into a for loop, it takes ownership of the vector.
The reason being that it will call `into_iter` on whatever is passed into the loop.
```rust
for i in v {// Note that this takes ownership of the Vec
    println!("{i}");
}
// And it is the same as doing this
for i in v.into_iter() {
}
```
If we want to not lose ownership of the vector, we can call the `iter` or `iter_mut` method to get a borrow on the data.
We can do the same by passing it in by reference.
```rust
for i in v.iter() {
    println!("{i}");
}

// Or access it via reference
for i in &v {
    println!("{i}");
}
```

And for fun, here is a rough picture of what a Vec looks like under the hood:
```rust
pub struct Vec<T> {
    buf: *mut T,
	capacity: usize,
	len: usize,
}
```

### Strings
Strings are the owned version of `&str` and represent UTF-8 character strings.
Due to the nature of UTF-8 be a variable length encoding, this is the definition of a Rust String:
```rust
pub struct String {
    vec: Vec<u8>,
}
```
This means that methods that operate on the strings generally use byte positions. So `len` will give you the String's length in bytes.
To get the number of characters in a string, I tend to use this trick:
```rust
let char_count = String::from("Hello, 世界").chars().count();
```

Much like Vecs and slices you can use a `&String` in functions that take a `&str`.

To create a String, you can use the `to_string` method implemented on most types, `String::from` as seen above, or as the result of the `format!` macro.
```rust
let one_string = 4.to_string();
let red_string = String::from("red string");
let blue_string = format!("{}", "blue);// Works the same as println!

```

## Enums, Matching, and Error Handling
### Enums
Enums can be one of two kind in Rust, either C-like enumerations or what are called Tagged Unions.
```rust
// Here is a simple enumeration
enum IpAddrKind {
    V4,
	V6,
}

// Here is a tagged union representing a singlely linked list
enum List<T> {
	Nil,
	Cons(T, Box<List<T>>)
}

// you can also put struct definitions in enums
enum BinaryTree {
    Leaf,
	Branch {
    	left: Box<BinaryTree>,
		right: Box<BinaryTree>,
	}
}
```

Two useful enums that is blessed by the Rust's compiler are called `Option<T>` and `Result<T,E>`.
Are defined as so:
```rust
pub enum Option<T> {
    Some(T),
	None,
}

pub enum Result<T, E> {
    Ok(T),
    Err(T),
}
```
They are blessed because how they come with the standard library, how the compiler treats them, and how you can construct/access its variants.
These types are primarily used in situations where a function could fail. Option stands in the place of null that most languages have.

Here is an example of a function in C called `atoi` that has some error handling issues.
```c
int atoi(const char* nptr);

```
```
RETURN VALUE
       The converted value or 0 on error.
```
What if the value of `nptr` is `"0"`?

We would have to check the string every time to know if there was an error or not. 
Rust provides `Option<T>` to solve this problem. 
By separating error states into their own values, we can now write a better `atoi` (maybe even with a better name).
```rust
pub fn atoi(number_str: &str) -> Option<i32>;
```

Result is useful for when you want to provide error information.
To improve the above example, we can change it to be a result to now have more error information.
```rust
pub fn atoi(number_str: &str) -> Result<i32, String>;
```

In order to access the data in an Option or Result, you will have to handle the error state in some way. The easiest way is to use `unwrap` which will
convert the error state into an uncatchable exception. However, you should never call this unless you either know that it will be a Some or Ok,
you are prototyping, or have reached a state that is impossible to recover from (in this case you should use `expect` and provide an error message).
```rust
// How you normally construct an enum
let protocol = IpAddrKind::V4;
let list = List::Cons(42, Box::new(List::Nil));

// How you can construct an Option
let maybe_int = Some(32);
let not_an_int = None;

```

In Rust, these are useful for modeling data where there are multiple variants of the same type.
Under the hood, the ones with data look like this:
```
+-----+----------------------------+
| Tag | The largest variant's size |
+-----+----------------------------+
```
This means that the program always knows what variant it is.

But isn't copying/moving all that data around somewhat expensive?

It can be but the Rust compiler can actually do some cool optimizations with this.
Since references can't be null, an `Option<&T>` gets converted into a simple pointer and any checks for Option's None variant is a simple null check.

Cool right?

### Matching
To make accessing enums, structs, and tuples easy, Rust has the the concept called Pattern Matching.
Most people are familiar with the ordering of data and the equality of data. Rust has the high level concept of the shape of data.
Rust also enforces that all variants must be handled. To do a catch all, use an underscore (`_`) or a variable name (`x`) as one of the branches.
Here is how you can match on the various datatypes in Rust:
```rust
struct Foo {
    bar: i32,
}

// matching structs
let Foo { bar } = Foo::new(10);

match Foo::new(10) {
    Foo { bar: 10 } => println!("10"),
    Foo { bar } => bar,
}

// matching enums
let Some(x) = function_that_can_fail() else {
	// To match on enum variants in a let binding, 
    // you either have to provide a default value or in this case,
    // throw an uncatchable exception.
    panic!("function failed");
};

let list = List::Cons(42, Box::new(List::Nil));
match list {
    List::Nil => println!("End of List"),
	List::Cons(value, tail) => {
    	// We can have blocks of code in matches too.
		println!("{}", value);
	}
}

// Here are some other things you can do with matches
let (x, y) = (42, "hello world")
match tuple {
    (3, "hello") => todo!(),// We can also match on strings
    (3, _) => todo!(),// Here we ignore the string part
	_ => todo!(), /* Here we match on any input, ignoring the tuple. 
                     We can also put in a variable instead to get access
                     to the tuple. */
}
```

### Error Handling
Most useful programs will encounter at least some form of error at some point. 
Especially if they interact with the operating system in any way.
There are 2 ways of doing errors in Rust. Using an error type like `Option<T>` or `Result<T,E>` or panicking.

#### Panicking
Panicking is Rust's way of crashing your thread and freeing up resources. 
It is an uncatchable exception.
These should be used whenever your program enters into a state that it cannot recover from.
The easiest way to panic is by calling the `panic!` macro. It can do message formating just like `println!` and `format!`.
There are also some useful macros that cause a panic but tell the compiler useful things.
The `todo!` macro tells the compiler that you aren't done implementing the code and it will disable some errors. If you hit a todo then it will panic.

Another useful panic macro is the `unreachable!` macro. This tells the compiler that this branch will not be reached under any means.
It is great when there is a state that you know is invalid, and if you are wrong, then you get a panic.

#### Option and Result
These types are used everywhere in Rust to handle errors. Doing a match on all of these types can get tedious fast.
Panicking is most likely the wrong thing to do. Luckily Rust has a way of propagating Options and Results through the try operator `?`.
Try allows the caller to pass the error state to their caller to handle it. This acts like a structured try/catch.

Doesn't this mean that main will have to handle every error state?

No, you can simply write main this way:
```rust
use std::io;

fn main() -> io::Result<()> {
    let mut buffer = String::new();
    
    io::stdin().read_line(&mut buffer)?;
    Ok(())
}
```

## Packages and Crates
Rust has a really powerful module system that can be confusing at first.
There is a cheat sheet that goes over it and can be found [here](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet).

By default, all functions, structs, and enums are private to their parent modules. You can make these public by preappending `pub` to their definitions.
Rust files are their own modules but you may create ones within files.
This is commonly used to create unit tests within the the file.
```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

To import a module, use the `use` keyword and the path to the module.
```rust
// importing a module
use std::hash;
// importing a module from local project
use crate::foo::bar
```

You can also import everthing from a module, several things from a module, or a specific thing from a module.
```rust
// everything
use foo::*
// several things
use std::{collections::HashMap, sync::{LazyLock, RwLock}};
// specific things
use std::hash::HashMap;
```

It is also possible to quantify a name so you don't have to import it. This is great when you have name collisions.
```rust
use foo;

fn bar() -> i32 {
    todo!()
}

fn main() {
    let x = foo::bar();
	let y = bar();
}
```

### External Packages
To add an external package, add the package and its version to the dependencies section of your project's Cargo.toml file.
```toml
[package]
name = "rand-test"
version = "0.1.0"
edition = "2024"

[dependencies]
rand = "0.8.5"
```
```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}

```

## Traits
Traits are a lot like interfaces but are much more powerful. You can define traits on any type that follows what is called the orphan rule.

**The orphan rule states that you can implement a trait on a type if either the crate defines the type or the crate defines the trait.**

This makes it useful for following the the Open Close principle of software engineering.

*Software entities should be open for extension, but closed for modification.*


Traits are declared as such:
```rust
pub trait Summary {
    fn summarize(&self) -> String;
    // You can put more methods here.
}
```

Here is how to implement a trait on the two structs `NewsArticle` and `Tweet`:
```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// Implementation Here
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

// Implementation Here
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

Sometimes in our code, we don't care what the type is, we just care that it implements an interface.
An example of this is how in Java we use `List<T>` instead of `ArrayList<T>` or `LinkedList<T>` when defining the interface for lists.

In Rust we have what are called existential types. They allow us to state:

*There exists a type, such that it has a trait interface xyz.*

This is powerful because we can now pass in a mixture of different types of data that all share the same interface.
```
let mut summaries: Vec<Box<dyn Summary>> = Vec::new();
summaries.push(Box::new(NewsArticle::new()));
summaries.push(Box::new(Tweet::new()));

for summary in summaries {
    println!("{}", summary.summarize());
}

```

The compiler has two ways of expressing this constraint.
One way is with the `dyn` keyword. This keyword requires that we pass the argument as a reference type.
```rust
pub fn notify(item: &dyn Summary) {
    println!("Breaking news! {}", item.summarize());
}

// You can also used owned references like Box<T> as so:
pub fn notify(item: Box<dyn Summary>) {
    println!("Breaking news! {}", item.summarize());
}
```
This can cause some problems though when trying to return a closure/lambda from a function.
To remedy this issue, you can use `impl` instead of `dyn` and in this case you don't need a reference.
```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

pub fn add_n(n: i64) -> impl Fn(i64) -> i64 {
    |x: i64| {
        x + n
    }
}
```

So why should I ever use `dyn` then if I can just use `impl`?

Impl isn't always existential and under the hood the compiler does know the actual type that is being passed in.
I say use `dyn` over `impl` unless you can't use `dyn`.

## Generics
Rust supports generics or parametric polymorphism.
This allows you to share code across different types.
To add generics, you wrap a capitalized variable name (or often single letter) in angle brackets. 
Then that goes at the end of the name of the function, struct, or enum.
Here is how you use generics for your functions datatypes, and methods.

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

pub struct Point<X, Y> {
    x: X,
    y: Y,
}

impl<X, Y> Point<X, Y> {
    fn x(&self) -> &X {
        &self.x
    }
}
```

You can also use traits in generics as constraints.
You add a colon after the name of the generic and put in a plus separated list of traits.
```rust
pub fn print<S: AsRef<str>>(string: S) {
    print!("{}", string.as_ref());
}

```
Sometimes specifying constraints this way can get long as shown below:
```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32;
```
You can also use a `where` clause when it gets too long:
```rust
fn some_function(t: &T, u: &U) -> i32 
where 
    T: Display + Clone,
    U: Clone + Debug
{ todo!() }
```


## Additional Resources on Rust
These are YouTube channels that go over ideas and concepts of Rust that I find to be helpful and might make more sense than me.

[No Boilerplate](https://www.youtube.com/watch?v=Q3AhzHq8ogs&list=PLZaoyhMXgBzoM9bfb5pyUOT3zjnaDdSEP)

[Logan Smith](https://www.youtube.com/@_noisecode/videos)
I highly recommend checking out Logan Smith's Cursed C++ Casts video.
