+++
title = 'Smalltalk'
date = 2024-10-02T22:34:29-06:00
draft = true
type = 'post'
+++



### Structures
A positive `isize` represents how many objects fit in the Object's Data.
A negative `isize` represents how many bytes fit in the Object's data.
```rust
type ObjectRef = usize;


struct Object {
    size: isize,
    class: ObjectRef,
    super_object: ObjectRef,
    members: *mut ObjectData,
}

struct ObjectData {
    ref_count: AtomicUsize,
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
* Stack
* StackFrame
* String
* Symbol

```rust

struct Class {
    symbol_to_method: HashMap<Symbol, usize>,
    methods: [Block],
}

enum BlockBody {
    Native,
    Bytecode(ByteArray),
}


struct Block {
    body: Native,
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
    stack: Stack,
}

struct Process {
    interpreter: Interpreter,
    future: Arc<dyn BoxedFuture>,
}

struct Stack {
    frames: StackFrame,
}

struct StackFrame {
    current_code: ByteArray,
    current_code_index: usize
    operand_stack: Vec<Object>,
}

struct StringObject {
    pointer: *const str,
}

struct Symbol {
    pointer: *const str,
}
```
