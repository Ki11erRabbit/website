+++
title = 'Reactive'
date = 2025-01-12T20:28:10-07:00
draft = true
type = 'post'
+++





## Design

This language will be structured around a priority queue and a threadpool.
The priority queue will hold messages that get consumed every tick. The tick is the heartbeat of the whole system. Each tick will release a "tick" message that may be listened for.
All messages currently in the queue will be processed in this tick. Any new messages will be put into a buffer that will be placed into the priority queue on the next tick.
Each message will be sent to every function that will listen for this message. The listeners are known statically.
Functions are called in a threadpool for speed. Messages will only be processed as there are threads. Therefore if there are only 4 threads and 5 listeners of a message, then the extra will be processed the second a thread is available.

All messages may have zero or more values associated with the message. Functions that respond to the message must have arguments that match the message values.
Functions/Closures may be passed around as parameters. This is to allow for callbacks that may be needed for algorithms that require external information from a subsystem.

In order to facilitate stateful computations. Functions are given two types of stateful variables, one for the function itself and one that may be shared with 1 or more functions. The one that is shared is protected with a Read Write Lock in order to prevent race conditions.
The shared state is to allow for stateful algorithms that are split into multiple functions because they rely on external information to proceed


## Structures
These all use Rust-like syntax

```rust
type Symbol = usize; // An index into a symbol table

type Reference = usize;
type Index = usize; // Represents and index into a table
type VTableIndex = usize; // Represents an index into a table of VTables

enum TypeTag {
    U8,
    U16,
    U32,
    U64,
    I8,
    I16,
    I32,
    I64,
    F32,
    F64,
    Char, //U32
    Object, // u64
    RawStr(Index), // Index into String Table
}

```


#### Class
```rust
struct Class {
    name: Symbol,
    parents: Vec<Symbol>, // Is mirrored in Object.parent_objects
    vtables: Map<(Symbol, Symbol), VTableIndex>, // Key is (Starting Parent, Particular Child), value is the vtable for that object. This allows for calling parent methods via super while still overloading parent vtables
    members: Vec<MemberInfo>,
    signals: Vec<SignalInfo>,
}

struct VTable {
    symbol_mapper: Map<Symbol, Index>,
    table: [VirtFunc]
}

struct VirtFunc {
    name: Symbol,
    value: VirtFuncValue,
    responds_to: Option<Symbol>,
    arguments: Vec<TypeTag>,
    ReturnType: TypeTag
}

enum VirtFuncValue {
    Builtin,
    Bytecode,
    Compiled,
}

struct MemberInfo {
    type_tag: TypeTag,
    name: Symbol,
}

struct SignalInfo {
    name: Symbol,
    static: bool,
    arguments: Vec<TypeTag>,
}

```

#### Object
```rust
struct Object {
    class: Symbol,
    parent_objects_size: usize,
    parent_objects: *mut [Reference], // Mirrors Class.parents
    children_size: usize,
    children: *mut[Reference],
    data: [u8], // Binary data
}
```

#### Bytecode
```rust

type TypeTag = u8;
type BlockId = usize;

enum Bytecode {
    Nop,
    Breakpoint,
    LoadU8(u8),
    LoadU16(u16),
    LoadU32(u32),
    LoadU64(u64),
    LoadI8(i8),
    LoadI16(i16),
    LoadI32(i32),
    LoadI64(i64),
    LoadF32(f32),
    LoadF64(f64),
    Pop,
    Dup,
    Swap,
    StoreLocal(u8)
    LoadLocal(u8)
    StoreArgument(u8)
    Add,
    Sub,
    Mul,
    Div,
    Mod,
    SatAdd,
    SatSub,
    SatMul,
    SatDiv,
    SatMod,
    And,
    Or,
    Xor,
    Not,
    AShl,
    LShl,
    AShr,
    LShr,
    Neg,
    Equal,
    NotEqual,
    Greater,
    Less,
    GreaterOrEqual,
    LessOrEqual,
    Convert(TypeTag),
    BinaryConvert(TypeTag),
    CreateArray(TypeTag),
    ArrayGet(TypeTag),
    ArraySet(TypeTag),
    NewObject(Symbol),
    GetField(Symbol, Symbol, usize), // Class name, Class name, Member. The second Class name is to allow for selecting the particular parent to access the field.
    SetField(Symbol, Symbol, usize), // Class name, Class name, Member. The second Class name is to allow for selecting the particular parent to access the field.
    IsA(Symbol),
    InvokeVirt(Symbol, Symbol, Symbol), // Class Name, Class Name, Function Name. The two class names allow for calling super methods as well as overridden super methods
    InvokeVirtTail(Symbol, Symbol, Symbol), // Class Name, Class Name, Function Name. The two class names allow for calling super methods as well as overridden super methods
    EmitSignal(Symbol, Symbol), // Class Name, Signal Name
    EmitStaticSignal(Symbol, Symbol), // Class Name, Signal Name
    ConnectSignal(Symbol, Symbol, Symbol, Symbol), // Signal Name, Class Name, Class Name, Method Name. The top two stack values are used for this. The top object is connected to the bottom object's signal via the 2nd and 3rd Class Names + the Method Name
    Return,
    ReturnVoid,
    StartBlock(usize),
    Goto(BlockId), // Offset to next block from current block
    If(BlockId, BlockId),
    Switch(Vec<BlockId>, BlockId),
    
}
    
```
