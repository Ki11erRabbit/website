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
    table: Vec<RealSymbol>,
    map: HashMap<String, usize>,
}

struct RealSymbol {
    name: String,
    parent: usize,
    data_size: usize,
    /// object is the object to respond to, message is the message ref,
    /// receiver is a ref of who to send the message to,
    /// outmessage is a ref to the message to send out,
    /// response is a ref to the message to send back to the sender of message.
    handle_message: fn(object: usize, message: usize, receiver: *mut usize, outmessage: *mut usize, response: *mut usize, context: *mut Context),
}
```
##### Object Table
A table that contains all live objects. Since there is no way of knowing if an object is collectable in Smalllang, it is required for the programmer to provide a way to free objects. The indices into the table are the references used throughout the program.
```rust
struct ObjectTable {
    table: Vec<ObjectHeader>,
}

struct ObjectHeader {
    global: bool,
    object: *mut Object,
    symbol: usize,
}
```

##### Module Description
This is what is filled out by the `bootstrap` function to describe a module from a dynamic link library.
```rust
fn bootstrap(module: *mut Module) {}

struct Module {
    provided_symbols: *const CString,
    provided_symbols_len: usize,
    initialize_function: CString,
    symbols: *const Symbol,
    symbols_len: usize,
}

struct Symbol {
    name: CString,
    parent_name: CString,
    data_size: usize,
    handle_message_function: CString,
}

fn initialize(symbol_table_offset: usize, ctx: *mut Context) {}
```
The `provided_symbols` is a list of CStrings that represent C symbols that need to be loaded in order for the code to function. This includes things like functions and globals.

The `initialize_function` is a CString that is the name of the initialize function. This function must be provided because it will allow for the module to know where it is in the global symbol table.

A Symbol tells the system how to contruct a given object. The `parent_name` is so that the system can look up the string and figure out the symbol that needs to be used for the actual symbol.

##### Context
This object's primary goal is to control access to the global structures.
```rust
struct Context {}

impl Context {
    fn allocate_object(&self, symbol_index: usize, symbol_offset: usize) -> usize {}

    fn deallocate_object(&self, reference: usize) {}

    fn get_object(&self, reference: usize) -> *mut Object {}

    fn create_message(&self, message: CString, args: *const (), args_size: usize) -> usize {}

    fn create_unknown_message(&self) -> usize {}

    fn send_message(&self, to: usize, message: usize) {}
}

```


#### Functions
