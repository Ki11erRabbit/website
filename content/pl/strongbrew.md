+++
title = 'Strongbrew'
date = 2024-10-31T10:23:54-06:00
draft = true
type = 'post'
+++


# Strong Brew

## Introduction
Strong Brew is a general purpose programming language with gradual dependent types.
Gradual dependent types should make the language be able to as expressive as possible while maintaining ease of use.

### Syntax
Syntax will be mostly Rust-like but here is some of the changes to the language.

##### Try keyword
Try is a new keyword that allows you to try an operation and convert any errors into a Result type.
It is similar to try/catch in other languages but the difference being that it converts an error (exception) into a value that can be used.

```
let arr = [1,2,3,4];
let result = try arr[5];
```

##### Types and Generics
Since we have dependent types, generics and type signatures can now contain expressions. This allows for much more powerful expressiveness and type safety.
Here is a common idiom where we have a finite set of some natural number. This allows us to create a value
```
struct Fin<End: Nat> {
    value: value < End,
}
```
