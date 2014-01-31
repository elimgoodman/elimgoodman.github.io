---
layout: page
title: Mesh
---
{% include JB/setup %}

#The Mesh VM

THIS IS NOT USELESS

We're going to start off with some really basic Mesh commands, to get you familiar with the concepts and syntax of the language. 

At its most basic, the Mesh language is designed to manipulate the state of the Mesh **VM**. The Mesh VM has 3 pieces of state:

- A symbol table, which maps symbol names to values or references
- An instruction pointer, which holds the next instruction to be executed
- A set of connections to the outside world called **channels**

Channels are how VMs do almost all of their work. If the Mesh VM is a virtual computer, you can think of its input and output channels as virtual devices, like network connections and the keyboard. A Mesh VM can *ask* for information from a particular channel, or *tell* a particular channel to do something.

Let's start off really simply. If we open a new Mesh project, we're presented with an empty REPL. Because this is a new project, the Mesh REPL is working on a blank VM. 

Let's just try evaluating some simple expressions:

```
1 --> 1:Int
"Hello world" --> "Hello World":String
1.3 --> 1.3:Float
True --> True:Boolean
[1, 2, 3] --> [1, 2, 3]:(List Int)
{1: "Hi"} --> {1: "Hi"}:(Map Int String)
```

From the above examples, we can see that the Mesh VM is type-aware - it can infer the types of different expressions. Additionally, we can see that it has no problem with higher-level literals like strings, lists, and maps. 

Remember - the primary purpose of the Mesh language is to manipulate the internal state of the VM. To that end, let's alter the symbol table first:

```
"Hello world" >> :symbols.message --> :symbols.message => "Hello world"
```

The '>>' operator means "move into". ":symbols" represents our VM's internal symbol table. The "." operator indicates a "slot" in an element (similar to the dot operator in Python or Javascript). So, all together, that statement means "Move the string 'Hello world' into the slot called 'message' in our internal symbol table."

```
message --> "Hello world"
```

Or:

```
message >> :symbols.other_message --> :symbols.other_message => "Hello world"
```

Or we can examine the symbol table of the VM directly:

```
:symbols
message => "Hello world"
other_message => "Hello world"
```

One thing to note is that all of the instructions we've been entering have been read and evaluated immediately by the Mesh REPL. However, sometimes it's useful to have the REPL read an instruction or expression, but *not* evaluate it. To do that, we can surround an instruction with backticks, and the REPL won't evaluate it immediately:

```
1 >> :symbols.temp --> :symbols.temp => 1
`1 >> :symbols.temp2` --> SymbolDefinitionInstruction
```

Now, if we look at our symbol table, we'll see that there's nothing in temp2 (show this). That's because that second instruction wasn't evaluated - it was just read. We'll see why this is useful shortly.

Now that we know how to read to and write from the symbol table, let's look at the instruction pointer. Just as ":symbols" represents the internal symbol tables, ":instructions" represents the instructions that a VM has loaded, and ":current_instruction" is the instruction that's next to be executed:

```
:instructions
[]

:current_instruction
None
```

As expected, we haven't added any instructions to our VM, so that's not very interesting. Let's add some instructions.

Remember, the things we've been typing into the REPL this whole time *are* instructions. Let's add an instruction to modify the symbol table:

```
`28 >> :symbols.age` >> :instructions
```

You'll note that we surrounded our instruction in backticks, since we didn't want that instruction evaluated immediately. Let's see what's in our instruction set now:

```
:instructions

[28 >> :symbols.age]
```

There's our instruction!



