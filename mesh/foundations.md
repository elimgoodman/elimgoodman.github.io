---
layout: page
title: Mesh - Foundations
---
{% include JB/setup %}

Hi! I'm going to telling you about Mesh, a new way to build computer programs. You can think of Mesh like an IDE and a language smushed together. It draws a lot of inspiration from Smalltalk, Clojure, Emacs, and many other programming technologies.

I'm going to start off by talking about the fundamental ideas that Mesh builds on top of. Once we have a good understanding of the foundational concepts, we can start talking about some of the more unique aspects of the system.

Let's start off by asking a big, crazy question: what is a computer program? In Mesh, the program itself is represented as a data structure. You can think of it like an object in a traditional OO language - it has internal state (which is basically the variables, data structures and so forth have been added to the program), and methods (which are basically manipulations you can make to the program - adding a new method, say, or renaming a variable). As opposed to manipulating text directly, in Mesh, you tell the program object to undergo a series of discrete transformations. We call these transformations **motions**.

As an example, let's add a new function to our empty project. Also, keep in mind that we'll be doing things the hard way while we're still explaining these ideas - it would be crazy to actually program like this all the time!

Here's an empty project - not very exciting, is it?

![](/assets/mocks/output.png)

Let's open the command bar by hitting ":". Don't worry too much about the syntax of the command for right now:

![](/assets/mocks/output2.png)

And voila! Our new function appears:

![](/assets/mocks/output3.png)

```
 	($cursor.currentPackage).(addRegion! {{Fn}})
```

In addition to telling Mesh to perform motions, we can also ask for certain information about the state of our program (which is what that ($cursor.currentPackage) business is all about). For example:

```
	(count $regions)
	(count ($regions.filter ($cursor.currentPackage).contains))
```

Motions like these are how all programming happens in Mesh, though not usually from the command bar, as we've been doing it. For example, here's the syntax for adding a statement directly beneath the cursor:

```
	($cursor.currentRegion)
		.(addStatementAt!
			:idx => ($cursor.getCurrentStatementIdx),
			\:statement => \{\{Statement :type => StatementTypes.Define, :data => {:symbol => "...", :expr => "..."} }})
```
So, yeah, we're not gonna be doing too much of that. Instead, we bind common motions to keys. So, to perform that same motion, we simply hit "o", like in vim. Mesh than needs a bit more information to complete the motion, so it prompts us for it. Much easier!

One thing to note - even though the program is represented and manipulated like a structured object, it's still represented on disk as text files. So Mesh programs can still work with text-based tools like grep and git.

Now that we've got this idea of motions, it's probably helpful to talk about the "vocabulary" of motions that Mesh understands. We can think of this vocabulary in terms of nouns and verbs - the things inside the program that can be acted upon, and the actions that can be taken.

//I think concepts make value sets redundant

The top-level "nouns" of Mesh should be pretty familiar to most programmers - they are functions, structs, interfaces, value sets, and packages. Functions are imperative sequences of statements; structs describe how data in the program is structured, as well as methods that can be taken on data with that shape; interfaces describe general properties of different data structures; you can think of value sets as a mix of Java's enums and Haskell's value constructors; and packages group all of these guys together. These pieces should be relatively unsurprising. There are also two additional nouns that are a little more "meta", and those will be discussed in the next video.

It's also probably worth noting that there are some more mundane nouns that we won't talk about, such as statements, expressions, and symbols.

The verbs of the system are a little more interesting. One cool thing about Mesh is that all of the nouns in the Mesh environment are represented internally as instances of Mesh objects. So, if we want to know what changes we can make to the program, we can ask the pieces what they know how to do. Let's ask the current package what commands it understands:

```
	($cursor.currentPackage).(understoodMessages)
```

We can tell the current package to perform one of the actions it understands:

```
	($cursor.currentPackage).(setName! "my_new_package")
```

And there it is! In addition to representing the components of the program as Mesh objects, the motions that we use to modify and interrogate those objects are written as Mesh expressions. This allows us to perform some pretty powerful motions:

```
	($regions.filter #(eq (:type %) Fn))).(pluck :name).(append! "Modified")
```

So now we've appended the word Modified to every function in our program. While this might seem like a trivial example, hopefully this is beginning to give you an idea of what Mesh enables you to do.

