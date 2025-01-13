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
