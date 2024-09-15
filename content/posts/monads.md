+++
title = 'Monads'
date = 2024-09-14T16:21:22-06:00
type = 'post'
+++

# So what is a Monad?
A monad is a monoid in the category of endofunctors, what's the problem?

This definition is not very helpful to anyone really unless you understand each part.
However, even then, it still won't be helpful in the context of Computer Science.

## So what is it?
If you go looking for definitions you will find a definition that one of my professors told us:

A monad describes computation and monadic operations are able to sequence those steps.

Now that we have a clear definition of a monad, we can now begin to explain their usefulness.

## Identity Monad
Lets start with the identity monad. This monad simply wraps a value into a monad. Sounds useless, but it does has its uses. We won't be going
into those uses but instead use it as a stepping stone for ideas. The identity monad can be used to describe a sequence of effect-less computation.
This simple monad allows us to describe what does a sequence look like.

So this C code here:
```c
void do_things() {
    2 + 2;
    4 + 4;
    int x = 4 + 5;
    x + x;
}
```

Can be written as this Haskell code:
```haskell
do_things :: Identity Void
do_things = Identity (2 + 2) >>= (\_ -> Identity (4 + 4) >>= (\_ -> Identity (4 + 5) >>= (\ x -> Identity (x + x) >>= (\ _ -> Void))))
```

Haskell can become quite verbose when using sequences of bind and introduces do notation to simplify things.
```haskell
do_things :: Indentity Void
do_things = do
	_ <- Identity (2 + 2)
	_ <- Identity (4 + 4)
	x <- Identity (4 + 5)
	_ <- Identity (x + x)
	Identity Void
```

As you can see, it looks a lot more like the above C code.

The bind operation (>>=) isn't named well to explain what it does. Rust's equivalent is called `and_then` which makes sense because it is sequencing
computation.

We use the Identity monad to represent simple sequences of computation.
Now lets move onto more complicated monads.

## Maybe (Option) and Either (Result) as well
The maybe monad and the Either monad are often used to signify the possibility of failure in a computations. We will now be using a custom Rust-like
language to describe definitions.
```rust
enum Maybe<T> {
    Just(T),
    Nothing,
}
```
Just means we have a value and Nothing means that we didn't get value. Some like to think of it as Null like in other languages but it is much more
powerful. Since monads allow for sequencing, we can abstract away all the boilerplate of checking for null.

Rust has a similar notation to do in Haskell. You can use the question mark operator (?) to allow for an early return there was a Nothing.
So to use what we have learned we can now write a save division function that doesn't crash our program if we attempt to perform division by zero.
```rust
fn safe_div(x: i64, y: i64) -> Option<i64> {
    match y {
        0 -> None,
        y -> Some(x / y),
    }
}
```
We can now easily sequence divisions that could go wrong. Or more generally, we can now easily sequence computations that could go wrong.
Here we use the function we defined.
```rust
fn many_div(start: i64, list: &[i64]) -> Option<i64> {
	if list.len() == 0 {
    	Some(start)
	} else {
    	many_div(safe_div(start, list[0]?), list[1..])
	}
}
```
We use recursion for simplicity rather than loops so that we can easily show the chaining of computations.
The question mark causes an early return if we ever divide by zero.


## Async/Await/Promises
A promise is indeed a monad, unlike what this [website](https://buzzdecafe.github.io/2018/04/10/no-promises-are-not-monads) states just
not Javascript Promises.
In order to access an async value from a promise you need a callback, or what academia calls a continuation. When we do a bind operation, we pass in
our callback to continue our computation. So in this case Promises represent computations that haven't occured yet. Bind in this case ensures that
our computation only continues once the unfinished computation finishes. Once again monads represent sequenced compuations.


## State/Reader/Writer
If you look at the implementation of these monads, they basically use callbacks (continuations) to rewrite your code to sequence everything.

## Monads Everywhere
One downside to monads is that they infect everything. The function coloring problem is real and is annoying. Another downside is that
monads don't compose well with differing monads. For that problem you need special transformers. A development that can fix the coloring problem and
composition problem is algebraic effects. But that is for another post. But for now check out the
[Koka language](https://koka-lang.github.io/koka/doc/index.html) for a programming language that
can do algebraic effects.
