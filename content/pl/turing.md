+++
title = 'Turing'
date = 2024-09-21T19:23:37-06:00
type = 'post'
+++


# Ideas
A language like BrainF\*\*k and Forth.

Forth allows you to control a stack machine with some high level control. BrainF\*\*k allows you to control a Turing Machine but with extreme low level control. I have the idea make a high level BrainF\*\*k

## Limitations
It can be configured to use different sizes of numbers but by default, 64bit numbers will be used


## Syntax

#### Numbers
Writing down a number moves the read head by one and puts it on the tape.

#### Strings
Writing down a string will move each character onto the tape in order. On numerical sizes of less than 32bits, the bytes will be written instead.

#### Conditional
These constructs will read the current value from the read head and if it is non-zero, then the if will be executed, otherwise if there is an else branch, then that will be executed.
```
if <commands> end

if <commands> else <commands> end
```

#### Moving the Tape Head
There are two ways to move the head, one is to just move it by one increment, the other is to move it by a certain amount.
```
# Single
# Left
<
# Right
>

# Read from the head and move that amount
# Left
<|
# Right
|>

# Move A certain amount specified by a number
# Left 10
<10)
# Right 10
(10>
```

#### Arithmetic
We can do basic arithmetic like all other languages like adding, subtracting, multiplying, dividing, and modulo.
The semantics are just slightly different. You need to provide a direction and the operator.
The new value is written to the slot on the tape whichever way the arrow is facing.
A number can be put in between the two characters and cause the tape head to perform that operation with the current value and move that distance before completing the operation.
```
# Addition
# To the Left
<+
# To the Right
+>

# Multiplication
# To the Left
<*
# To the Right
*>

# Subtraction
# To the Left
# Semantics: A - B = A <- B
<-
# To the Right
# Semantics: B - A = A -> B
->

# Division
# To the Left
# Semantics: A / B = A </ B
</
# To the Right
# Semantics: B / A = A /> B

# Modulo
# To the Left
# Semantics: A % B = A <% B
<%
# To the Right
# Semantics: B % A = A %> B
%>

# Moving Arithmetic
# Semantics [A B C D] A + D = A +3> B
# To the Left
<3+
# To the Right
/4>
```

#### Loops
There is still a need for loops despite the existance of conditionals and unconditional jumps.
Thus there are two loop constructs.
While will read from the tape head and if it is non-zero then it will execute the commands. Otherwise it will skip them.
Loop will execute the commands and then check if the tape head has a non-zero value. If not, then the next command will be executed
```
# While
while <commands> end

# Loop
loop <commands> end
```

#### Output
A period will cause the current tape head value to be outputed to the console.

A comma will cause the current tape head value to to interpreted and outputed as utf32. If bit size is less than 32bits then the tape head will output and possibly move for utf16 and utf8 encodings

#### Functions
A function can be called in two different ways. One way is to call the function with a command.
The other is to call a function by its reference on the tape.
Functions don't take any parameters.
```
# Declaring a function
fun <name> <commands> end.

# Putting a function onto the tape.
getfun <name>

# Calling a funtion from the tape.
call

```


