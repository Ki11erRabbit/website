+++
title = 'Smalltalk'
date = 2024-10-02T22:34:29-06:00
draft = true
type = 'post'
+++



### Structures

```rust
type ObjectRef = usize;


struct Object {
    ref_count: AtomicUsize,
    size: isize,
    class: ObjectRef,
    super_object: ObjectRef,
    members: *mut ObjectData,
}

struct ObjectData {
    members: [ObjectRef]
}

```

#### Special Objects
* Class
* Block
* ByteArray
* Char
* File
* Float
* Integer
* Interpreter
* Process
* String
* Symbol

```rust

struct Class {
    methods: [Block],
}

enum BlockBody {
    Native,
    Bytecode,
}


struct Block {
    body: [BlockBody]
}

struct ByteArray {
    bytes: [u8],
}

struct Char {
    character: char,
}

struct File {
    //whatever is needed for files
}

struct Float {
    value: f64,
}

struct Integer {
    value: i64,
}

struct Interpreter {
    // Whatever is needed for the interpreter
}
```
