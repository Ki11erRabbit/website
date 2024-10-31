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
Here is a common idiom where we have a finite set of some natural number. This allows us to create a natural number that is has an upper bound that is enforced by the typesystem.
```
struct Fin<End: Nat> {
    value: value < End,
}
```

##### Function Calling Syntax
Functions can be called the traditional way or they may be called with a dot syntax.
The dot syntax will work if the type of the first parameter and the expression on the left side of the dot matches. This should allow for discoverability of functions.
Parenthesese may also be dropped for consiseness.

```
"Hello, World!".print

```

##### Closure/Lambda Syntactic sugar.
Taking an idea from Koka, a closure may be put outside of the closing parenthesis. This makes the nesting of symbols reduced, improving readability.
Another idea from Koka, is that if a closure doesn't take any parameters then the parameter list may be excluded for readability.
This should also allow for generalizing and abstracting processies making the language much smaller and more maintainable.
```
[1,2,3,4].map() |x| x + 1;

let x = 2;
while {x < 10} {
    x += 1;
};

```
Yes, `while` is a function.
* Wouldn't this slow down the language?
Maybe, but I am making the bet that the JVM will optimize out the function call at runtime, making the call free. Even if it doesn't the JVM should be fast enough as is to make it worth it.


### Grammar
##### Keywords
These are the keywords of the language. There should be a minimal amount of keywords in order to keep the language small.

```
// Control Flow
if
else
match
fn
return
break
continue
// Data
struct
enum
// Variable
let
mut
const
// Module
import
module
// Visibility Modifier
pub
// External Code Modifier
extern
// Core Types
i8
i16
i32
i64
int
nat
f32
f64
char
char32
```

### Core Spec
This defines the functions and types that are built into the language itself and not the standard library

##### Core Functions
```
pub fn print(str: String);

pub extern "Java" fn while(cond: f() -> bool, body: f()) """
    while (cond.run()) {
        body.run()
    }
"""


// There will be one for each numeric type
pub extern "Java" fn range/for(start: int, end: Int, body: f(int)) """
	for (Int x = start; x.isLessThan(end); x = x.increment()) {
    	body.run(x);
	}
"""

// There will be one for each numeric type
pub extern "Java" fn range/for-while<A>(start: int, end: int, body: f(int)-> Maybe<A> ) -> Maybe<A> """
	Maybe<A> output = new Nothing<>();
	for (Int x = start; x.isLessThan(end) && output.isNone(); x = x.increment()) {
    	output = body.run(x);
	}
	return output;
"""

pub extern "Java" fn repeat(amount: nat, action: f()) """
	for (Nat x = Nat.zero(); x.isLessThan(amount); x = x.increment()) {
    	action.run();
	}
"""


```


