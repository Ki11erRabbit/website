+++
title = 'Learning_rust2'
date = 2025-02-06T20:58:41-07:00
type = 'post'
+++

This is adapted from the BYU CS 465 website which can be found [here](https://cs465.zappala.org/winter2025/notes/learning-rust-2/)

I will be going in a slightly different order than the ubove webpage, but I think it will be infomative to all.

## Owned vs Unowned Values
Due to Rust's ownership rules, there are many types that represent these rules. Here are a few of them.

|Name              |Unowned| Owned  |
|------------------|-------|--------|
|Reference (Borrow)|`&T`   |`Box<T>`,`Rc<T>`,`Arc<T>`|
|Slice (Array)     |`&[T]` or [T;n] |`Vec<T>`|
|UTF-8 Character String|`&str` |String  |

Many owned values contain heap allocated values but will automatically clean up their resources when they go out of scope.

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

v = v.into_iter().map(|x| x + 1).collect::<Vec<_>>();

for i in v {// Note that this takes ownership of the Vec
    println!("{i}");
}
// To borrow the contents of the Vec, user iter or iter_mut

for i in v.iter() {
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

## Enums and Matching
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

A useful enum that is blessed by the Rust's compiler is called `Option<T>`.
It is defined as so:
```rust
pub enum Option<T> {
    Some(T),
	None,
}
```
It is blessed because of how you can construct/access its variants among some other things that will be talked about later.
This type is primarily used in situations where a function could fail. This stands in the place of null that most languages have.
In order to access the data in an Option, you will have to handle the error state in some way. The easiest way is to use `unwrap` which will
convert the error state into an uncatchable exception. However, you should never call this unless you either know that it will be a Some,
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
	// To match on enum variants in a let binding, you either have to provide a default value or in this case, throw an uncatchable exception.
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
	_ => todo!(), // Here we match on any input, ignoring the tuple. We can also put in a variable instead to get access to the tuple.
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

