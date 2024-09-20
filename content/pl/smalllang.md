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
    handle_message: fn(object: *mut Object, message: *const Object) -> *const Object
}

```

### Bytecode
While there is bytecode, it isn't executed. Instead it is JIT compiled to make everything Faster.
```rust
enum Bytecode {
    Halt,
    Nop,
    PushNull,
	Pop,
	Dup,
	Swap,
	StoreLocal(u8),
	LoadLocal(u8),
    CreateObject(usize),
    SendMessage(usize, usize),
    StartBlock(usize),
    GotoBlock(usize),
    IfBlock(usize, usize),
}
```
|Code             |   Description|       u16         |
|:----------------|:-------------|:-----------------:|
|Halt|Suspends execution|0x00|
|Nop|Does nothing but increment the program counter|0x01|
|PushNull|Pushes Null onto the Stack |0x02|
|Pop|Pops a value off of the operand stack|0x03|
|Dup|Duplicates the top value off of the operand stack and pushes it onto the stack|0x04|
|Swap|Swaps the top two values of the operand stack|0x05|
|StoreLocal|Stores the top value off of the operand stack to the specified index|0x06|
|LoadLocal|Pushes the value at the specified index onto the operand stack|0x07|
|CreateObject|Creates an object from the specified index into the symbol table and pushes it onto the operand stack|0x08|
|SendMessage|Creates a message object with the message name being an index into the symbol table and the amount of arguments off of the stack and sends it to the value off the top of the stack.|0x09|
|StartBlock|Indicates the start of a codeblock for Cranelift|0x0A|
|GotoBlock|Goes to a block specified by an index|0x0B|
|IfBlock|Goes to either block depending if the top value on the stack is zero or not.|0x0C|
