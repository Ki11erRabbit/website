+++
title = 'Smalllang'
date = 2024-09-19T19:08:03-06:00
type = 'post'
+++


# What is Smalllang?
Smalllang is a Smalltalk inspired language with the goal of being incredibly minimal. So minimal that nothing is implemented.

Why?

So that the language can be grown however the programmer/user sees fit. This also leaves the implementation to be much simpler and therefore easier
to maintain.


# Specification
## High Level Overview
Mark and Sweep Garbage Collected, JIT compiled, Reflexive, Dynamic, Object Oriented



## Low Level Overview
Many of the specification will be in pseudo-Rust code.

### Object Structure
An object is just a VTable with the underlying data being the Rust equivalent to C's `Void*`.
There is a vtable that as of now only holds the `handle_message` function which will execute code depending on what the message passed in was.
There is also a parent pointer to allow for object inheritance.

```rust
struct Object {
	vtable: *const VTable,
	parent: *mut Object,
	data_size: usize,
	data: *mut (),
}

struct VTable {
    handle_message: fn(object: usize, message: usize, receiver: *mut usize, outmessage: *mut usize, response: *mut usize)
}

```
The `handle_message` function is how an object can respond to messages. The `usize`s the object's references that can be used to access an object from the global object table.
It takes the object ref and message ref as well as a pointers that indicate who to send a new message to, what the message ref is, and a message ref to send back to the sender of the message.


### Runtime

#### Data Structures
##### Symbol Table
A table that contains all symbols of the current runtime system. When a module is initialized, it is given an offset into this table which will be where the symbols added will start at.
```rust
struct SymbolTable {
    table: Vec<Symbol>,
}
```
##### Object Table
A table that contains all live objects. Since there is no way of knowing if an object is collectable in Smalllang, it is required for the programmer to provide a way to free objects. The indices into the table are the references used throughout the program.
```rust
struct ObjectTable {
    table: Vec<*mut Object>,
}
```


#### Functions
