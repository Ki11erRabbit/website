+++
title = 'Shell Ideas Part 1'
date = 2024-09-16T18:44:45-06:00
type = 'post'
+++

I have an idea for a shell language that is Unix api inspired. In my mind a shell should primarily do these things:
1. Provide an easy way to run commands
2. Provide easy access to IO streams
3. Be interactive.

Most shells fulfill two of these requirements, those being the easy of running commands and interactive.

But isn't it easy to pipe between commands? Yes, but it gets complicated if you want to duplicate a stream and send it between two commands. You also cannot send a stream into a function. What I mean for #2 is that streams are first class structures in a script.

## What I propose in a shell to do such a thing
I propose that streams (pipes, sockets, etc) can be manipulated freely just like variables.

This is the syntax I suggest for manipulating streams:
```
# accessing a specific stream
@0 #stdin
@1 #stdout
@2 #stderr

# Stream capturing
# That is, converting a stream into a string
@{cat text.txt}

# Stream Redirection
# Redirecting a stream to another one
@1>@2

# Stream Binding
# That is, creating a variable that is a stream
@hello=echo hello

# Accessing Bound Streams
# That is, accessing a stream variable and passing it into a pipeline.
@hello | grep e

```

And to allow this we need to make the following changes to the shell language.
```
# Variable Binding
$hello='hello'

# Variable expansion
'${hello}'

```

