---
layout: page
title: Mesh - The Language And Terms
---
{% include JB/setup %}

#The Mesh Language

##Statements
Mesh is a language that's comprised of lists of statements. The general form for statements is **$statement_type** *$args*. Each statement type takes different arguments. Here are all of the statements that Mesh understands:

- **def** *$symbol_name* *$expr*: adds a new symbol to the symbol table. Note that once a symbol has been added to current symbol table via **def**, it cannot be redefined.
- **ref** *$symbol_name* *$expr*: very similar to def, ref adds an entry to the symbol that *can* be redefined (via **alter**, below).
- **alter** *$ref* *$expr*: sets the symbol *$ref* equal to *$expr*. 
- **perform** *$expr*: performs *$expr*. This is used to invoke functions whose only purpose is to cause side-effects.
- **log** *$expr*: very similar to *$expr*, this is also used to perform side-effect-causing functions. However, there are greater limitations on what the side effects can be (can't read from IO, must fail naively, etc), which means that **log** statements are sometimes permitted in contexts that might otherwise require pure code.
- **loop** *$expr* *$iterator* *$block*: 
- **loop** *$expr* *$iterator* *$block*: 

#Common Terms

- **Function**:
- **Struct**:
- **Interface**:
- **Value Set**:
- **Concept**:
- **Region**:
- **Perspective**:
- **Motion**:
- **Trigger**:
- **VM**:
- **Channel**: